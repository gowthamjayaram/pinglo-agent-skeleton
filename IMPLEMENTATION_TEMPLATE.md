# Agent Implementation Template

Build agents in 6 phases. Complete each phase before moving to next.

---

## Phase 1: Agent Core Logic

**Deliverable:** `agents/base_agent.py` with agent loop

**Checklist:**
- [ ] Define tool schema (receive_message)
- [ ] Write run_agent(context) function
- [ ] Claude API call with tools
- [ ] Tool execution logic
- [ ] Error handling for tool failures
- [ ] Unit test: agent uses correct tool
- [ ] Unit test: agent handles tool timeout
- [ ] Unit test: agent terminates (doesn't loop forever)

**Gates:**
- [ ] All unit tests pass
- [ ] No exceptions raised to Claude

---

## Phase 2: Agent Integration with Triggers

**Deliverable:** `triggers/` with use case handlers

**Checklist:**
- [ ] Write inactivity_trigger.py (detects stale users)
- [ ] Write activity_reminder_trigger.py (detects non-responders)
- [ ] Trigger calls run_agent() with context
- [ ] Integration test: trigger → agent → tool call

**Gates:**
- [ ] Integration tests pass

---

## Phase 3: Configuration & Environment

**Deliverable:** `config.py` with env vars

**Checklist:**
- [ ] PINGLO_MCP_SERVER_URL from env
- [ ] CLAUDE_API_KEY from env
- [ ] Defaults documented
- [ ] No hardcoded values

**Gates:**
- [ ] Config loads from .env

---

## Phase 4: Error Handling & Resilience

**Deliverable:** Tests for failure paths

**Checklist:**
- [ ] Test: MCP server timeout
- [ ] Test: MCP server 404
- [ ] Test: MCP server 500
- [ ] Test: Network error
- [ ] Test: Claude API error
- [ ] All errors return structured response

**Gates:**
- [ ] All error tests pass

---

## Phase 5: Documentation

**Deliverable:** Updated docs

**Checklist:**
- [ ] Update CLAUDE.md with current status
- [ ] Add inline code comments
- [ ] Update README.md

**Gates:**
- [ ] All docs match implementation

---

## Phase 6: E2E Testing

**Deliverable:** E2E tests with real Pinglo MCP

**Checklist:**
- [ ] Test: inactivity_agent with real MCP
- [ ] Test: activity_reminder_agent with real MCP
- [ ] Test: agent handles MCP responses correctly

**Gates:**
- [ ] All E2E tests pass
- [ ] All phases 1-5 complete
