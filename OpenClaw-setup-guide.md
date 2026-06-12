# OpenClaw Setup Guide
## School Management · HR · Business Systems
### Large Team (10+) · SQL Server · Web App Automation

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Core Configuration](#core-configuration)
4. [Skills to Install](#skills-to-install)
5. [School Management System](#school-management-system)
6. [HR System](#hr-system)
7. [Business System](#business-system)
8. [Team Channel Setup](#team-channel-setup)
9. [Scheduled Automations](#scheduled-automations)
10. [Security Tips](#security-tips)

---

## Overview

OpenClaw connects your three systems — School Management, HR, and Business — to a single AI agent that your entire team can interact with via messaging apps (Telegram, Slack, WhatsApp). It can:

- **Monitor** dashboards and alert on anomalies
- **Automate** form fills, report generation, and data sync
- **Learn** your codebase and generate documentation
- **Query** your SQL Server database in plain English

---

## Prerequisites

- Node.js 18+
- OpenClaw v3.2 or later
- SQL Server (any edition)
- A running OpenClaw Gateway instance
- Telegram, Slack, or WhatsApp for team channels

Install OpenClaw:

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

---

## Core Configuration

Add this to `~/.openclaw/openclaw.json`:

```json
{
  "mcpServers": {
    "mssql": {
      "command": "npx",
      "args": ["-y", "mcp-node-mssql"],
      "env": {
        "DB_HOST": "your-server",
        "DB_PORT": "1433",
        "DB_USERNAME": "your-username",
        "DB_PASSWORD": "your-password",
        "DB_DATABASE": "your-database",
        "DB_TRUST_SERVER_CERTIFICATE": "true",
        "CONNECTION_TIMEOUT": "600000",
        "REQUEST_TIMEOUT": "300000"
      }
    }
  }
}
```

Restart the gateway after saving:

```bash
openclaw restart
```

---

## Skills to Install

Install all four skills your systems need:

```bash
# Browser automation — screenshots, form fills, monitoring
openclaw skills install browser-automation

# SQL toolkit — query, summarize, report from SQL Server
openclaw skills install sql-toolkit

# Documentation — learn and map your codebase
openclaw skills install documentation

# Scheduler — recurring automated tasks
openclaw skills install cron
```

Verify installation:

```bash
openclaw skills list
```

---

## School Management System

### Monitor & Screenshots

```
# Daily attendance dashboard screenshot → send to admin group
Every day at 7:30am → screenshot https://school-app/admin/attendance → send to @school-admins

# Alert if enrollment drops
Every Monday → query enrollments table → if count < last_week alert @principal
```

Example SQL query to ask OpenClaw:

```
"Summarize total enrolled students by grade level for this semester"

"Which students have more than 3 unexcused absences this month?"

"Show me fee payment status — who is overdue by more than 30 days?"
```

### Automate Tasks

```
"Fill the new student registration form at https://school-app/register
 with data from the students_pending table"

"Generate a monthly attendance report for all classes and send to
 the teachers group on Telegram"

"Send fee reminder messages to all parents with outstanding balance"
```

### Learn the Codebase

```
"Read all files in /school-app/src and document every module —
 enrollment, grading, timetable, fees, and user management"

"Map all database tables in the school DB and show their relationships"

"List all API endpoints in the school management system"
```

---

## HR System

### Monitor & Screenshots

```
# Payroll dashboard screenshot on pay day
Every 25th of month at 9am → screenshot https://hr-app/payroll → send to @hr-team

# Leave balance alert
Every Monday → query leave_balances table → alert if any employee < 2 days remaining
```

Example SQL query to ask OpenClaw:

```
"Show headcount by department as of today"

"Which employees have contracts expiring in the next 30 days?"

"Generate monthly payroll summary broken down by department"

"List all employees who haven't completed their onboarding checklist"
```

### Automate Tasks

```
"Fill the leave approval form for all pending requests in the
 leave_requests table where status = 'pending'"

"Generate offer letters for all new hires added this week
 and send to the HR manager"

"Send contract renewal reminders to employees expiring next month"
```

### Learn the Codebase

```
"Document the full HR system codebase — modules include:
 recruitment, onboarding, payroll, leave, appraisal, offboarding"

"Map the employee lifecycle from hire to termination
 across all HR database tables"

"Show me all stored procedures in the HR database"
```

---

## Business System

### Monitor & Screenshots

```
# Daily KPI dashboard → WhatsApp business group
Every day at 8am → screenshot https://biz-app/dashboard/kpi → send to @management

# Low stock alert
Every hour → query inventory table → alert if any item stock < reorder_level
```

Example SQL query to ask OpenClaw:

```
"Show this week's revenue vs target by product category"

"Which customers haven't placed an order in the last 60 days?"

"Summarize monthly expenses vs budget for each department"

"Show top 10 products by sales volume this quarter"
```

### Automate Tasks

```
"Generate weekly sales report and send to the management group every Friday at 5pm"

"Create purchase orders for all inventory items below reorder level"

"Sync new employee headcount from HR system into the business
 budget planning table"
```

### Learn the Codebase

```
"Read /business-app/src and document all modules:
 finance, inventory, sales, reporting, and user management"

"Map how the business system connects to the HR and
 school management systems via shared database tables or APIs"

"List all scheduled jobs and background tasks in the business system"
```

---

## Team Channel Setup

For a large team, give each department its own channel so OpenClaw responses are targeted.

### Telegram Setup (Recommended)

```
@school-admins   → School Management alerts + reports
@hr-team         → HR alerts, payroll reports, leave summaries
@management      → Business KPIs, revenue reports, inventory alerts
@it-team         → Codebase docs, error alerts, deployment status
```

### Slack Setup

```
#school-management   → School alerts and reports
#hr-system           → HR automation and summaries
#business-ops        → Business KPIs and inventory
#openclaw-alerts     → All critical system alerts in one place
```

Connect a channel in OpenClaw:

```bash
openclaw channels add telegram --token YOUR_BOT_TOKEN --chat-id YOUR_CHAT_ID
openclaw channels add slack --webhook YOUR_SLACK_WEBHOOK
```

---

## Scheduled Automations

Add these to your `~/.openclaw/schedule.md` or via chat:

```
# School
Every day 7:30am   → attendance screenshot          → @school-admins
Every Monday 8am   → weekly enrollment summary      → @school-admins
Every month 1st    → fee collection report          → @principal

# HR
Every day 9am      → headcount by department        → @hr-team
Every 25th 9am     → payroll dashboard screenshot   → @hr-team
Every Monday       → contracts expiring this month  → @hr-manager

# Business
Every day 8am      → KPI dashboard screenshot       → @management
Every Friday 5pm   → weekly sales report            → @management
Every hour         → inventory low stock check      → @warehouse-team

# IT / Codebase
Every Sunday 10pm  → generate updated docs for      → @it-team
                     all three systems
```

---

## Security Tips

- Never hardcode credentials — always use environment variables
- Run OpenClaw in an isolated environment, not your primary workstation
- Use read-only DB credentials for reporting tasks; separate write credentials for automation tasks
- For browser automation, use the Extension Relay mode only for trusted internal workflows
- Rotate API keys every 90 days
- Bind your SQL Server to `127.0.0.1` if running locally:

```yaml
# docker-compose.yml
ports:
  - "127.0.0.1:1433:1433"
```

---

## Quick Reference: Example Prompts by System

| System | Example Prompt |
|---|---|
| School | "How many students are enrolled this semester by grade?" |
| School | "Which teachers have not submitted grades for Term 2?" |
| HR | "List employees with contracts expiring in 30 days" |
| HR | "Generate payroll summary for October" |
| Business | "Show revenue vs target for Q3 by department" |
| Business | "Which inventory items are below reorder level?" |
| All | "Document the codebase for [system name]" |
| All | "Screenshot the dashboard and send to the team" |

---

*Generated with OpenClaw Setup Guide · June 2026*
