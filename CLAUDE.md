# Pinglo Agent Skeleton Development Guide

## User Preferences

### Communication Style
- Reply with minimum number of sentences
- When writing code or copious information: get explicit permission first
- No unnecessary preamble or explanations

---

## Project Summary

Demonstrates agentic architecture using Claude API. Agents invoke Pinglo MCP Server tools to send context-aware messages.

---

## Architecture

```
Agent Skeleton
├── Agents (Claude API loop)
│   ├── base_agent.py → tool calling loop
│   ├── inactivity_agent.py → re-engage users inactive > 7 days
│   └── activity_reminder_agent.py → remind tagged but non-responding users
├── Triggers (query Pinglo DB)
│   ├── inactivity_trigger.py → find stale users
│   └── activity_reminder_trigger.py → find non-responders
└── Tests
```

**Configuration:**
- `PINGLO_MCP_SERVER_URL` — env var (default: `http://localhost:8000`)
- `CLAUDE_API_KEY` — env var

**Dependencies:**
- Pinglo MCP Server (configurable URL)
- Claude API
- requests, anthropic

---

## How They Connect

Trigger → Agent receives context → Agent calls Claude → Claude calls Pinglo MCP tools → Result → Agent finishes

---

## Development

See `IMPLEMENTATION_TEMPLATE.md` for phase-by-phase workflow.
