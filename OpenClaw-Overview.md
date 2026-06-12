![OpenClaw](files/openclaw.jpeg)

# OpenClaw Overview

OpenClaw is essentially a self-hosted AI agent platform. What it can do depends on the skills, tools, and integrations you enable, but here is a comprehensive overview.

## Messaging & Communication

OpenClaw can connect to many chat platforms through a single gateway, including:

- Telegram
- WhatsApp
- Discord
- Slack
- Signal
- iMessage
- Google Chat
- Microsoft Teams
- Matrix
- LINE
- Twitch
- Web Chat
- Many others through plugins and extensions

---

## AI Assistant Tasks

- Answer questions
- Hold conversations
- Remember context
- Multi-agent workflows
- Route tasks to different AI models
- Long-running conversations
- Shared or isolated workspaces

---

## Automation

- Run scheduled jobs (cron)
- Monitor inboxes
- Process messages automatically
- Trigger actions based on events
- Execute workflows
- Autonomous background tasks

---

## Web & Research

- Search the web
- Read websites
- Gather information
- Compare products and services
- Research topics
- Summarize documents
- Monitor updates

---

## File Operations

- Read files
- Create files
- Edit documents
- Organize folders
- Manage local data
- Work with PDFs and other documents

---

## Computer & Server Control

- Execute shell commands
- Run scripts
- Manage Linux servers
- Automate deployments
- Perform maintenance tasks
- Interact with local applications

---

## Media Processing

- Analyze images
- Process audio
- Handle video
- Voice transcription
- Text-to-speech
- Image generation
- Video generation

---

## Mobile Device Features

### Android Node

- Pair Android devices
- Voice chat
- Camera access
- Device commands
- Mobile notifications

### iPhone Node

- Voice interaction
- Camera access
- Screen recording
- Location access
- Mobile workflows

---

## Integrations

OpenClaw can integrate with:

- Gmail
- GitHub
- Calendar systems
- Cloud storage
- Smart home devices
- Databases
- APIs
- Custom services

---

## For Developers

- Use OpenAI models
- Use Claude models
- Use Gemini models
- Use Grok models
- Use local Ollama models
- Use self-hosted models
- Build custom skills
- Create plugins
- Create custom workflows

---

# AI Provider Layer

## Supported Providers

### OpenAI
- **Models:** GPT-4o, GPT-4.1, GPT-5

### Anthropic
- **Models:** Claude Sonnet, Claude Opus

### Google
- **Models:** Gemini Pro, Gemini Flash

### DeepSeek
- **Models:** DeepSeek Chat, DeepSeek Reasoner

### Ollama (Local Models)
- **Supported Models:** Llama, Gemma, Mistral, Qwen, DeepSeek

---

## Things to Be Careful About

Because OpenClaw can:

- Execute commands
- Access files
- Control tools
- Store credentials

It should be configured carefully and kept updated. Security researchers have highlighted risks involving malicious skills and agent permissions if not properly managed.


# OpenClaw + SQL Server Guide

## 1. Summarize / Report on Data

Once connected, just message OpenClaw naturally:

- "Summarize orders by region for Q2 2026"
- "Show me month-over-month revenue growth"
- "Which customers haven't ordered in 90 days?"

It inspects your schema first, then generates and runs the query, returning results as a formatted summary — no manual SQL needed.

---

## 2. Write SQL Queries

Since you're advanced, OpenClaw can handle the heavy stuff:

- CTEs, window functions, dynamic pivots
- Stored procedure drafts
- Index optimization suggestions
- Query rewrites for performance

Just describe what you need: *"Write a query ranking sales reps by monthly revenue using a window function, partitioned by region."*

---

## 3. Automate Scheduled Tasks

This is where OpenClaw really shines. You can set up recurring jobs like:

```
Every Monday 8am   → run summary report        → send to Telegram/WhatsApp
Every night        → check for anomalies        → alert me if found
Every hour         → sync new orders            → to a reporting table
```

---

## Setup: Connect SQL Server via MCP

Add the following to `~/.openclaw/openclaw.json`:

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

Then restart the OpenClaw gateway — it will auto-discover your SQL Server tools.

> **Security tip:** Never hardcode credentials. Use environment variables and restrict your DB host to `127.0.0.1` if running locally.

