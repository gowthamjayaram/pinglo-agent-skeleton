# Product Canvas — Inactivity Agent

**Canvas version:** 1.0
**PM:** Gowtham
**Date:** 2026-03-09
**Status:** Draft

---

## 1. Headline

Pinglo autonomously nudges users who haven't interacted for 7+ days with a friendly "missing you" message via agentic re-engagement.

---

## 2. Problem Statement

**The User** struggles with dormant engagement in Pinglo because after 7+ days of inactivity, they forget about the app entirely — the relationship goes cold. Without automated re-engagement, churn increases and the user never returns to coordinate activities.

---

## 3. Target Users

| Persona | Impact |
|---|---|
| User (v0) | Primary — receives friendly nudge to re-engage after inactivity |
| Circle (future v1) | Secondary — entire group nudged if no one has coordinated activity |

---

## 4. Strategic Positioning

- **Why now:** Agentic foundation is built (agent loop, Claude API, Pinglo MCP tools). Re-engagement is a high-impact use case demonstrating agent autonomy.
- **Strategic fit:** Agents are the next frontier of Pinglo. This feature validates that agents can reason about user state and take proactive action.
- **Risk of skipping:** Without agent demonstration, agentic vision remains theoretical. This feature surfaces real challenges (state detection, personalization, timing).

---

## 5. User Stories & Acceptance Criteria

### Story 1: Agent detects and nudges inactive user
> As an **Agent**, I want to detect users inactive 7+ days and send them a friendly re-engagement message, so that dormant users are reminded about Pinglo.

**Acceptance criteria:**
- [ ] Given a user with `last_interaction_timestamp < now - 7_days`, when the inactivity trigger fires, then agent receives context: `{"trigger": "inactivity", "user_id": "...", "days_inactive": X}`
- [ ] Given inactivity context, when agent calls Claude, then Claude decides to send a message
- [ ] Given Claude's decision, when agent calls `receive_message` tool, then message is sent via Pinglo
- [ ] Given message sent successfully, when tool returns `{"success": true}`, then agent finishes cleanly

### Story 2: Agent handles errors gracefully
> As an **Agent**, I want to handle errors when a user doesn't exist or message fails, so that transient errors don't crash the agent.

**Acceptance criteria:**
- [ ] Given a user_id that doesn't exist, when agent calls receive_message tool, then tool returns `{"success": false, "error": "..."}`
- [ ] Given tool failure, when agent processes the error, then agent logs and finishes (no retry)

---

## 6. Feature Scope

### In scope
- Trigger detects users with `last_interaction_timestamp > 7_days`
- Agent receives inactivity context via `run_agent(context_dict)`
- Agent calls Claude API with inactivity context + tool definitions
- Claude reasons about situation and decides whether to send message
- Agent calls `receive_message` tool via HTTP POST to Pinglo MCP Server
- Agent handles structured error responses from tool
- Full test coverage: unit + integration + E2E

### Out of scope
- Circle-level inactivity (v1 feature)
- Configurable inactivity threshold (hardcoded to 7 days for v0)
- Scheduled triggers (v0: manual invocation only)
- Message personalization beyond "missing you" template
- Retry logic or exponential backoff
- Analytics or nudge effectiveness tracking

### Assumptions & Technical Dependencies
- Pinglo MCP Server is running on `localhost:8000`
- `receive_message` tool is available and working
- Claude API is accessible with valid `CLAUDE_API_KEY`
- Pinglo backend returns users with `last_interaction_timestamp` field
- Agent context is provided by trigger (no direct DB query in agent)

---

## 7. Success Metrics

- **Agent completes flow:** Agent receives context → calls Claude → calls tool → finishes (v0 success = 100% happy path)
- **Tool integration:** `receive_message` successfully delivers message to Pinglo backend
- **Error handling:** All error paths tested and logged without crashing agent
- **Code quality:** ≥80% test coverage on agent logic
