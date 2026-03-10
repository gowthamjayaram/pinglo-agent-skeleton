# Tech Canvas — Inactivity Agent

**Canvas version:** 1.0
**Engineer Lead:** TBD
**Date:** 2026-03-10
**Status:** Ready for Phase 1

---

## 1. Architecture Overview

```
Trigger (inactivity_trigger.py)
    ↓ (detects inactive users, provides context)
Context: {"trigger": "inactivity", "user_id": "...", "days_inactive": 7+}
    ↓
Agent Loop (inactivity_agent.py)
    ├─ Send context + tools to Claude API
    ├─ Claude reasons: "Should I nudge this user?"
    ├─ Claude calls receive_message tool
    └─ Agent executes tool via Pinglo MCP Server (HTTP POST)
        ↓
Pinglo MCP Server (localhost:8000)
    ├─ Receive tool call
    ├─ Validate message + user_id
    └─ Send message to user
        ↓
User sees friendly nudge message in Pinglo
```

**Key insight:** Agent is a decision-maker, not an executor. Trigger detects, Claude decides, MCP Server executes.

---

## 2. Agent Loop Design

### Input Context
Trigger provides inactivity context to agent:
```python
{
    "trigger": "inactivity",
    "user_id": "user-123",
    "days_inactive": 9,
    "last_interaction_date": "2026-03-01T10:30:00Z",
    "user_circle_names": ["Friends", "Work"]  # optional: for future v1 personalization
}
```

### Claude API Configuration
- **Model:** `claude-3-5-haiku-20241022` (fast, cost-effective for simple decisions)
- **Max tokens:** 256 (agent only needs to decide + call tool, no long reasoning)
- **Temperature:** 0.7 (slight creativity for friendly messages, not deterministic)
- **Tools:** `receive_message` (1 tool only for v0)

### Tool Definitions
```json
{
    "name": "receive_message",
    "description": "Send a friendly re-engagement message to an inactive user via Pinglo",
    "input_schema": {
        "type": "object",
        "properties": {
            "user_id": {
                "type": "string",
                "description": "The Pinglo user ID to send message to"
            },
            "message": {
                "type": "string",
                "description": "Friendly re-engagement message (e.g., 'We're missing you! Come back to plan activities')"
            }
        },
        "required": ["user_id", "message"]
    }
}
```

### System Prompt (Context for Claude)
```
You are the Pinglo re-engagement agent. Your job is to help users who've been inactive
reconnect with the app and their communities.

Given an inactive user, your job is to:
1. Understand how long they've been inactive
2. Compose a brief, friendly message that makes them want to return
3. Send the message via the receive_message tool

Keep messages:
- Short (1-2 sentences max)
- Warm and genuine (not salesy or pushy)
- Action-focused ("Come back to plan activities with your friends")
- Avoid FOMO or guilt-tripping

Example good message: "Hey! We're missing you. Check back to coordinate activities with your circles."
Example bad message: "You haven't used Pinglo in 9 days. Our app is better without inactive users."
```

### Agent Loop Termination
Agent terminates when:
- `stop_reason == "end_turn"` (Claude decides no tool call needed)
- Tool `receive_message` executes and returns (Claude is done)
- Max iterations = 3 (safety limit to prevent infinite loops)

---

## 3. Key Components

### Trigger (triggers/inactivity_trigger.py)
**Responsibility:** Detect inactive users and provide context to agent

**Input:**
- Pinglo backend user data (user_id, last_interaction_timestamp, user_circles)
- Inactivity threshold: 7 days (hardcoded in v0)

**Process:**
```python
def get_inactive_users(days_threshold=7):
    """Query Pinglo backend for inactive users"""
    # GET /users/?last_interaction=< now - 7_days
    # Returns: [{"user_id": "...", "last_interaction_timestamp": "...", "circles": [...]}, ...]

def create_inactivity_context(user):
    """Format user data into context for agent"""
    return {
        "trigger": "inactivity",
        "user_id": user["user_id"],
        "days_inactive": (now - user["last_interaction_timestamp"]).days,
        "last_interaction_date": user["last_interaction_timestamp"],
        "user_circle_names": [c["name"] for c in user.get("circles", [])]
    }
```

**Output:** List of context dicts for agent to process

**Error handling:**
- If backend query fails: Log error, skip batch (don't crash trigger)
- If user doesn't exist: Include in results anyway (agent will handle gracefully)

### Agent (agents/inactivity_agent.py)
**Responsibility:** Receive inactivity context, call Claude, execute tool

**Input:** Context dict from trigger

**Process:**
```python
def run_inactivity_agent(context):
    """
    1. Validate context
    2. Send to Claude API with tools
    3. Loop: if Claude calls tool, execute it
    4. Return result
    """
    # Step 1: Validate context
    validate_inactivity_context(context)  # Ensure user_id, days_inactive present

    # Step 2: Send to Claude
    messages = [
        {
            "role": "user",
            "content": f"""
            An inactive user needs re-engagement:
            - User ID: {context["user_id"]}
            - Days inactive: {context["days_inactive"]}
            - Last interaction: {context["last_interaction_date"]}
            - Circles: {", ".join(context.get("user_circle_names", []))}

            Should I send them a friendly message to return to Pinglo?
            """
        }
    ]

    # Step 3: Loop
    for iteration in range(3):
        response = claude.messages.create(
            model="claude-3-5-haiku-20241022",
            max_tokens=256,
            tools=[RECEIVE_MESSAGE_TOOL_SCHEMA],
            messages=messages
        )

        if response.stop_reason == "end_turn":
            # Claude decided no message needed (shouldn't happen, but handle)
            return {"success": True, "action": "skip", "reason": "Claude decided not to nudge"}

        if response.stop_reason == "tool_use":
            # Claude wants to call receive_message tool
            tool_call = extract_tool_call(response)
            result = execute_receive_message(tool_call["user_id"], tool_call["message"])

            # Loop back with tool result
            messages.append({"role": "assistant", "content": response.content})
            messages.append({"role": "user", "content": f"Tool result: {result}"})
        else:
            break  # Unexpected stop reason

    return {"success": True, "message_sent": True}
```

**Output:**
```python
{
    "success": True,
    "user_id": "user-123",
    "message": "We're missing you! Come back to plan activities",
    "tool_result": {"delivered": True, "timestamp": "..."}
}
# or
{
    "success": False,
    "user_id": "user-123",
    "error": "Tool execution failed",
    "error_details": "..."
}
```

**Error handling:**
- Invalid context: Validate early, raise error
- Claude API error: Catch, log, return error dict
- Tool execution fails: Log, return error (don't retry)
- Max iterations reached: Log warning, return partial result

### Tests

#### Unit Tests (tests/agents/test_inactivity_agent.py)
```python
# Mock Claude responses

def test_agent_calls_receive_message_with_valid_context():
    """Agent receives context, calls Claude, extracts tool call"""

def test_agent_handles_claude_api_error():
    """If Claude API fails, agent returns error dict gracefully"""

def test_agent_handles_tool_execution_failure():
    """If receive_message tool fails, agent logs and finishes cleanly"""

def test_agent_validates_context_early():
    """If context missing user_id or days_inactive, agent fails fast"""

def test_agent_terminates_after_tool_execution():
    """After tool call succeeds, agent stops (no infinite loop)"""

def test_agent_respects_max_iterations():
    """If Claude loops > 3 times, agent stops with error"""
```

#### Integration Tests (tests/integration/test_inactivity_agent_integration.py)
```python
# Real Claude API, mocked Pinglo MCP Server

def test_full_inactivity_workflow():
    """Trigger → Agent → Claude → Tool call → Message sent"""

def test_agent_with_real_claude_decision_making():
    """Real Claude reads inactivity context, decides to nudge, calls tool"""

def test_agent_handles_missing_user_gracefully():
    """If user_id doesn't exist in Pinglo, MCP tool returns error, agent handles"""

def test_agent_handles_network_timeout():
    """If Pinglo MCP Server is unreachable, agent catches and returns error"""
```

#### E2E Tests (tests/e2e/test_inactivity_agent_e2e.py)
```python
# Real Claude API, Real Pinglo MCP Server, real tool execution

def test_complete_inactivity_flow():
    """Full flow: trigger detects user → agent receives context → Claude decides →
    message delivered to Pinglo → user sees message"""
```

---

## 4. Dependencies

### External
- **Claude API** (via Anthropic SDK)
  - Endpoint: `https://api.anthropic.com/v1/messages`
  - Auth: `CLAUDE_API_KEY` env var
  - Rate limit: Depends on plan (monitor in production)

- **Pinglo MCP Server** (HTTP)
  - Endpoint: `http://localhost:8000/messages/` (configurable via PINGLO_MCP_SERVER_URL)
  - Method: POST
  - Expects: `{"user_id": "...", "message": "..."}`
  - Returns: `{"success": true/false, "message_id": "...", "error": "..."}`

### Internal
- **anthropic SDK** (latest stable)
- **agents/base_agent.py** (reusable agent loop, if applicable)
- **config.py** (env vars: CLAUDE_API_KEY, PINGLO_MCP_SERVER_URL, INACTIVITY_DAYS_THRESHOLD)
- **logging** (standard library, for error/info logs)

---

## 5. Error Handling Strategy

| Error Scenario | Agent Response | Logging |
|---|---|---|
| Invalid context (missing user_id) | Fail fast, raise ValueError | ERROR: "Invalid inactivity context: missing user_id" |
| Claude API auth error (invalid API key) | Return error dict, don't retry | ERROR: "Claude API auth failed: {details}" |
| Claude API rate limit | Return error dict, don't retry | ERROR: "Claude API rate limited" |
| Claude API timeout (>30s) | Catch timeout, return error | ERROR: "Claude API timeout after 30s" |
| Claude returns unexpected format | Log and fail | ERROR: "Claude response missing tool_use: {response}" |
| Pinglo MCP Server unreachable | Catch connection error | ERROR: "Pinglo MCP Server unreachable: {url}" |
| Pinglo MCP Server returns `{"success": false}` | Log, finish cleanly, don't retry | WARN: "Tool execution failed: {error}" |
| Max iterations reached (>3) | Log warning, return partial result | WARN: "Agent exceeded max iterations" |

**General principle:** Fail gracefully. Log everything. Never crash. Let trigger continue processing other users.

---

## 6. Configuration & Environment

```env
# .env (required for agent)
CLAUDE_API_KEY=sk-ant-v1-...
PINGLO_MCP_SERVER_URL=http://localhost:8000
INACTIVITY_DAYS_THRESHOLD=7
LOG_LEVEL=INFO
```

---

## 7. Testing Checklist

- [ ] Unit tests: Agent logic with mocked Claude (≥80% coverage)
  - [ ] Valid context → tool call
  - [ ] Invalid context → early fail
  - [ ] Claude API error → error handling
  - [ ] Tool failure → graceful exit
  - [ ] Max iterations → termination

- [ ] Integration tests: Real Claude, mocked tools (≥70% coverage)
  - [ ] Full workflow: context → Claude → decision → tool call
  - [ ] Missing user handling
  - [ ] Network timeout handling

- [ ] E2E tests: Real Claude, real Pinglo MCP
  - [ ] Complete flow end-to-end
  - [ ] Verify message actually delivered to Pinglo

- [ ] Error path coverage: ≥80% of error scenarios tested

- [ ] All tests passing before Phase 3 code review

---

## 8. Success Criteria

- ✅ Agent receives context, calls Claude, extracts tool call correctly
- ✅ Message delivered to Pinglo backend without errors
- ✅ All error paths handled gracefully (no crashes, all logged)
- ✅ Agent terminates after 1-3 iterations (no infinite loops)
- ✅ Test coverage ≥80% on agent logic
- ✅ Code review sign-off before Phase 3

