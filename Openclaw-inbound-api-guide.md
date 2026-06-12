# OpenClaw Inbound API Guide
## School Management · HR · Business Systems
### Calling OpenClaw from Your Web Applications

---

## Table of Contents

1. [How It Works](#how-it-works)
2. [Gateway Setup](#gateway-setup)
3. [Authentication](#authentication)
4. [Endpoint Reference](#endpoint-reference)
5. [School Management — Code Examples](#school-management--code-examples)
6. [HR System — Code Examples](#hr-system--code-examples)
7. [Business System — Code Examples](#business-system--code-examples)
8. [Hook Mappings Config](#hook-mappings-config)
9. [OpenAI-Compatible Endpoint](#openai-compatible-endpoint)
10. [Security Best Practices](#security-best-practices)

---

## How It Works

```
Your Web App (School / HR / Business)
        |
        | HTTP POST (Bearer Token)
        ▼
OpenClaw Gateway (port 18789)
        |
        | Processes prompt + tools
        ▼
SQL Server · Browser · Telegram/Slack
```

Your app fires an HTTP request to OpenClaw. OpenClaw receives it, runs the AI agent, queries your SQL Server or browser, and sends the result back — or pushes it to your team channel.

---

## Gateway Setup

Enable hooks in `~/.openclaw/openclaw.json`:

```json
{
  "gateway": {
    "port": 18789,
    "bind": "loopback"
  },
  "hooks": {
    "enabled": true,
    "auth": {
      "token": "your-secret-token-here"
    },
    "mappings": {
      "new-student":    { "action": "wake",  "template": "New student registered: {{name}}, Grade: {{grade}}" },
      "fee-payment":    { "action": "wake",  "template": "Fee payment received: {{student_name}} paid {{amount}}" },
      "leave-request":  { "action": "agent", "session": "hr-agent" },
      "new-employee":   { "action": "wake",  "template": "New employee onboarded: {{name}}, Dept: {{department}}" },
      "low-stock":      { "action": "wake",  "template": "Inventory alert: {{item}} has only {{quantity}} units left" },
      "monthly-report": { "action": "agent", "session": "biz-agent" }
    }
  }
}
```

Restart the gateway after saving:

```bash
openclaw restart
openclaw doctor
```

---

## Authentication

All requests require a Bearer token in the Authorization header.

```http
Authorization: Bearer your-secret-token-here
Content-Type: application/json
```

Generate a strong token:

```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

Store it as an environment variable — never hardcode it:

```bash
# .env
OPENCLAW_TOKEN=your-secret-token-here
```

---

## Endpoint Reference

| Endpoint | Method | Description |
|---|---|---|
| `/hooks/wake` | POST | Fire-and-forget trigger. Fast, no reply needed. |
| `/hooks/agent` | POST | Full agent run. Returns 202, processes async. |
| `/hooks/<name>` | POST | Named mapped hook with custom payload template. |
| `/v1/chat/completions` | POST | OpenAI-compatible. Use when you need a response back. |
| `/api/sessions/main/messages` | POST | Send message to main agent session. |

**mode options for `/hooks/wake`:**
- `"now"` — process immediately
- `"next-heartbeat"` — batch into next cycle (lighter on resources)

---

## School Management — Code Examples

### 1. New Student Registered → Notify Admin

Trigger this from your student registration form on save:

```javascript
// school-app/controllers/studentController.js

async function onStudentRegistered(student) {
  await fetch("http://localhost:18789/hooks/new-student", {
    method: "POST",
    headers: {
      "Authorization": `Bearer ${process.env.OPENCLAW_TOKEN}`,
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      name: student.full_name,
      grade: student.grade_level
    })
  });
}
```

OpenClaw receives this and automatically:
- Looks up the student record in SQL Server
- Sends a summary to the `@school-admins` Telegram group

---

### 2. Fee Payment Received → Update & Receipt

```javascript
// school-app/controllers/feeController.js

async function onFeePaymentReceived(payment) {
  await fetch("http://localhost:18789/hooks/fee-payment", {
    method: "POST",
    headers: {
      "Authorization": `Bearer ${process.env.OPENCLAW_TOKEN}`,
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      student_name: payment.student_name,
      amount: payment.amount
    })
  });
}
```

---

### 3. Ask OpenClaw a Question & Get a Response Back

Use this when your dashboard needs live data from OpenClaw:

```javascript
// school-app/controllers/reportController.js

async function getAttendanceSummary() {
  const response = await fetch("http://localhost:18789/v1/chat/completions", {
    method: "POST",
    headers: {
      "Authorization": `Bearer ${process.env.OPENCLAW_TOKEN}`,
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      model: "openclaw:main",
      messages: [
        {
          role: "user",
          content: "Summarize today's attendance by grade level from the SQL Server attendance table"
        }
      ]
    })
  });

  const data = await response.json();
  return data.choices[0].message.content;
}
```

---

### School Trigger Events Summary

| Event | Endpoint | OpenClaw Action |
|---|---|---|
| Student registered | `/hooks/new-student` | Record saved + notify admins |
| Fee payment received | `/hooks/fee-payment` | Update DB + send receipt |
| Exam results published | `/hooks/wake` | Screenshot results page + notify |
| Monthly report due | `/hooks/agent` | Generate report + send to group |

---

## HR System — Code Examples

### 1. Leave Request Submitted → Check Balance & Flag

```javascript
// hr-app/controllers/leaveController.js

async function onLeaveRequestSubmitted(request) {
  await fetch("http://localhost:18789/hooks/leave-request", {
    method: "POST",
    headers: {
      "Authorization": `Bearer ${process.env.OPENCLAW_TOKEN}`,
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      prompt: `Employee ID ${request.employee_id} submitted a leave request
               for ${request.days} days starting ${request.start_date}.
               Check their leave balance in SQL Server.
               If balance >= requested days, approve it and notify HR.
               If insufficient, flag it and alert the HR manager.`,
      session: "hr-agent"
    })
  });
}
```

---

### 2. New Employee Added → Start Onboarding

```javascript
// hr-app/controllers/employeeController.js

async function onNewEmployeeAdded(employee) {
  await fetch("http://localhost:18789/hooks/new-employee", {
    method: "POST",
    headers: {
      "Authorization": `Bearer ${process.env.OPENCLAW_TOKEN}`,
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      name: employee.full_name,
      department: employee.department
    })
  });
}
```

---

### 3. Get Payroll Summary for Dashboard

```javascript
// hr-app/controllers/payrollController.js

async function getPayrollSummary(month, year) {
  const response = await fetch("http://localhost:18789/v1/chat/completions", {
    method: "POST",
    headers: {
      "Authorization": `Bearer ${process.env.OPENCLAW_TOKEN}`,
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      model: "openclaw:main",
      messages: [
        {
          role: "user",
          content: `Generate payroll summary for ${month}/${year}
                    broken down by department from SQL Server payroll table`
        }
      ]
    })
  });

  const data = await response.json();
  return data.choices[0].message.content;
}
```

---

### HR Trigger Events Summary

| Event | Endpoint | OpenClaw Action |
|---|---|---|
| Leave request submitted | `/hooks/leave-request` | Check balance + approve/flag |
| New employee added | `/hooks/new-employee` | Start onboarding checklist |
| Contract expiring | `/hooks/wake` | Send renewal reminder |
| Payroll run | `/hooks/agent` | Generate payroll report + send |

---

## Business System — Code Examples

### 1. Low Stock Alert → Notify Warehouse

```javascript
// business-app/controllers/inventoryController.js

async function onLowStockDetected(item) {
  await fetch("http://localhost:18789/hooks/low-stock", {
    method: "POST",
    headers: {
      "Authorization": `Bearer ${process.env.OPENCLAW_TOKEN}`,
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      item: item.name,
      quantity: item.current_stock
    })
  });
}
```

---

### 2. Monthly Report Trigger → Send to Management

```javascript
// business-app/controllers/reportController.js

async function triggerMonthlyReport(month, year) {
  await fetch("http://localhost:18789/hooks/monthly-report", {
    method: "POST",
    headers: {
      "Authorization": `Bearer ${process.env.OPENCLAW_TOKEN}`,
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      prompt: `Generate the full business report for ${month}/${year}:
               1. Revenue vs target by department
               2. Top 10 products by sales
               3. Expense summary vs budget
               4. Headcount from HR system
               Send the report to the @management Telegram group.`,
      session: "biz-agent"
    })
  });
}
```

---

### 3. KPI Dashboard — Live Data from OpenClaw

```javascript
// business-app/controllers/dashboardController.js

async function getKPISummary() {
  const response = await fetch("http://localhost:18789/v1/chat/completions", {
    method: "POST",
    headers: {
      "Authorization": `Bearer ${process.env.OPENCLAW_TOKEN}`,
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      model: "openclaw:main",
      messages: [
        {
          role: "user",
          content: `Pull today's KPIs from SQL Server:
                    - Total revenue today vs target
                    - Orders count
                    - Inventory items below reorder level
                    - Active employees on leave today
                    Return as JSON.`
        }
      ]
    })
  });

  const data = await response.json();
  return JSON.parse(data.choices[0].message.content);
}
```

---

### Business Trigger Events Summary

| Event | Endpoint | OpenClaw Action |
|---|---|---|
| Low stock detected | `/hooks/low-stock` | Alert warehouse team |
| New order placed | `/hooks/wake` | Update inventory + confirm |
| Monthly report due | `/hooks/monthly-report` | Generate + send to management |
| Budget exceeded | `/hooks/wake` | Alert finance manager |

---

## Hook Mappings Config

Full `~/.openclaw/openclaw.json` hooks section for all three systems:

```json
{
  "hooks": {
    "enabled": true,
    "auth": {
      "token": "your-secret-token-here"
    },
    "mappings": {
      "new-student":    { "action": "wake",  "template": "New student registered: {{name}}, Grade: {{grade}}" },
      "fee-payment":    { "action": "wake",  "template": "Fee payment: {{student_name}} paid {{amount}}" },
      "leave-request":  { "action": "agent", "session": "hr-agent" },
      "new-employee":   { "action": "wake",  "template": "New employee: {{name}}, Dept: {{department}}" },
      "low-stock":      { "action": "wake",  "template": "Low stock: {{item}} has {{quantity}} units left" },
      "monthly-report": { "action": "agent", "session": "biz-agent" }
    }
  }
}
```

---

## OpenAI-Compatible Endpoint

Use `/v1/chat/completions` when your app needs a **response back** from OpenClaw (e.g. for dashboards, reports, or inline results).

```javascript
// Generic reusable helper for all three systems

async function askOpenClaw(prompt) {
  const response = await fetch("http://localhost:18789/v1/chat/completions", {
    method: "POST",
    headers: {
      "Authorization": `Bearer ${process.env.OPENCLAW_TOKEN}`,
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      model: "openclaw:main",
      messages: [{ role: "user", content: prompt }]
    })
  });

  const data = await response.json();
  return data.choices[0].message.content;
}

// Usage examples
const attendance = await askOpenClaw("Summarize today's attendance by grade");
const payroll    = await askOpenClaw("Generate payroll summary for this month");
const kpi        = await askOpenClaw("Show revenue vs target for this week");
```

---

## Security Best Practices

- **Never hardcode tokens** — always use environment variables
- **Use HTTPS in production** — expose OpenClaw behind an Nginx reverse proxy with SSL
- **Verify request signatures** — use HMAC-SHA256 to confirm requests came from your app
- **Bind to loopback** — keep gateway on `127.0.0.1`, never expose port 18789 directly to the internet
- **Use separate tokens per system** — school, HR, and business apps each get their own token
- **Rotate tokens every 90 days**

HMAC signature verification example:

```javascript
const crypto = require("crypto");

function verifySignature(payload, receivedSig, secret) {
  const expected = crypto
    .createHmac("sha256", secret)
    .update(JSON.stringify(payload))
    .digest("hex");
  return crypto.timingSafeEqual(
    Buffer.from(expected),
    Buffer.from(receivedSig)
  );
}
```

---

*Generated with OpenClaw Inbound API Guide · June 2026*
