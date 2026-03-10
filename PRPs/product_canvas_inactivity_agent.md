# Product Canvas — Inactivity Agent

**Canvas version:** 1.0
**PM:** Gowtham
**Date:** 2026-03-10
**Status:** Ready for Implementation

---

## 1. Headline

Pinglo autonomously nudges dormant users with a friendly re-engagement message to bring them back and reignite activity coordination.

---

## 2. Problem Statement

**The User** struggles with forgotten engagement because after several days of inactivity, they lose the habit of checking Pinglo — the app fades from memory. Without proactive nudging, users churn and never return to coordinate activities with their circles.

---

## 3. Target Users

| Persona | Impact |
|---|---|
| User (v0) | Primary — receives friendly nudge after 7+ days of inactivity |
| Circle (future v1) | Secondary — entire group nudged if no one has coordinated activity |

---

## 4. Strategic Positioning

- **Why now:** Agent foundation is built (agent loop, Claude API, Pinglo MCP tools). Re-engagement is a high-impact use case demonstrating agent autonomy and personalization.
- **Strategic fit:** Agents are the next frontier of Pinglo. This feature validates that agents can reason about user state and take proactive action without explicit triggers.
- **Risk of skipping:** Without demonstrating agent autonomy, the agentic vision remains theoretical. This feature surfaces real challenges: state detection, timing, personalization, and error resilience.

---

## 5. User Stories & Acceptance Criteria

### Story 1: Inactive user receives friendly re-engagement message
> As a **User** who hasn't used Pinglo in 7+ days, I want to receive a friendly nudge, so that I'm reminded to return and coordinate activities.

**Acceptance criteria:**
- [ ] Given a user has not interacted with Pinglo for 7 or more days, when the inactivity re-engagement runs, then the user receives a message in Pinglo
- [ ] Given the user receives the message, then the message is friendly and encouraging (e.g., "We're missing you! Come back to plan activities with your circles")
- [ ] Given the user returns after receiving the nudge, then they can resume coordinating activities as normal

### Story 2: Re-engagement handles delivery failures gracefully
> As a **User**, I want the re-engagement system to be reliable, so that failures don't silently prevent nudges or break the app.

**Acceptance criteria:**
- [ ] Given a message fails to deliver to a user, when the system detects the failure, then it logs the error and continues operating (no crash)
- [ ] Given a user ID doesn't exist or is invalid, when re-engagement is attempted, then the system gracefully handles it without sending a message
- [ ] Given the re-engagement system encounters an error, then the system continues to monitor other inactive users without interruption

---

## 6. Feature Scope

### In scope
- Detect users with `last_interaction > 7 days` (configurable in implementation)
- Agent receives inactivity context via trigger
- Agent calls Claude to compose a friendly message
- Agent sends message via Pinglo MCP Server `receive_message` tool
- Graceful error handling for missing users, network failures, or API errors
- Full test coverage: unit + integration + E2E

### Out of scope
- Circle-level inactivity detection (v0: User only)
- Configurable inactivity threshold in UI (v0: hardcoded to 7 days)
- Scheduled/recurring triggers (v0: manual invocation or basic scheduler)
- Message personalization beyond "missing you" template (future enhancement)
- Retry logic or exponential backoff (v0: fail fast, log, continue)
- Analytics or nudge effectiveness tracking (future feature)
- User preferences to opt out of nudges (v1 feature)

### Assumptions & Technical Dependencies
- Pinglo MCP Server is running and accessible
- `receive_message` tool is available and working
- Claude API is accessible with valid `CLAUDE_API_KEY`
- Pinglo backend returns users with `last_interaction_timestamp` field
- Agent context is provided by trigger (no direct DB query in agent)
- Trigger logic will detect and provide inactive user data to agent

---

## 7. Success Metrics

- **Happy path completion:** Agent receives inactivity context → calls Claude → delivers message → finishes cleanly (100% of test cases)
- **Tool integration:** `receive_message` successfully delivers message to Pinglo backend
- **Error resilience:** All error paths tested (missing users, network timeouts, API errors) and handled without crashing agent
- **Code quality:** ≥80% test coverage on agent logic (unit + integration)
- **User experience:** Inactive user receives message and understands the nudge is genuine (qualitative feedback)
