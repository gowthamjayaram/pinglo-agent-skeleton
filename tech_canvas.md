# Tech Canvas Template — Agent Feature

Use this template to spec the technical implementation of each agent feature.

---

## 1. Architecture Overview

Diagram showing:
- Agent loop components
- Claude API interaction
- Tool calls to Pinglo MCP Server
- Data flow

---

## 2. Agent Loop Design

### Input Context
What information does the trigger provide to the agent?
```python
{
    "trigger": "inactivity" | "activity_reminder" | ...,
    "user_id": "...",
    "additional_data": {...}
}
```

### Claude API Configuration
- Model: `claude-3-5-haiku-20241022`
- Max tokens: [X]
- Tools: [list of tools agent will define]

### Tool Definitions
```json
{
    "name": "receive_message",
    "description": "...",
    "input_schema": {...}
}
```

### Agent Loop Termination
What signals completion? (stop_reason == "end_turn" or max iterations reached)

---

## 3. Key Components

### Trigger (triggers/[feature]_trigger.py)
- Input: [Query logic or event]
- Output: context dict for agent
- Error handling: [how failures are handled]

### Agent (agents/[feature]_agent.py)
- Input: context dict
- Process: Claude API call → tool execution → result handling
- Output: final result or error
- Error handling: [catch tool failures, API errors, etc.]

### Tests

#### Unit Tests
- Mock Claude responses
- Test: agent uses correct tool
- Test: agent handles tool failure
- Test: agent terminates

#### Integration Tests
- Real Claude API (with mocks for tool execution)
- Full agent workflow

#### E2E Tests
- Real Claude API
- Real Pinglo MCP Server
- Real tool invocation

---

## 4. Dependencies

### External
- Claude API (CLAUDE_API_KEY)
- Pinglo MCP Server (PINGLO_MCP_SERVER_URL)

### Internal
- `agents/base_agent.py` — reusable agent loop
- `config.py` — configuration management
- `anthropic` SDK

---

## 5. Error Handling Strategy

| Error Type | Agent Response |
|---|---|
| Tool returns `{"success": false}` | Log and finish (don't retry) |
| Claude API error | Return error dict to caller |
| Network timeout | Catch and return structured error |
| Invalid context | Validate early, fail fast |

---

## 6. Testing Checklist

- [ ] Unit tests for agent with mocked Claude
- [ ] Integration tests with real Claude, mocked tools
- [ ] E2E tests with real services
- [ ] Error path coverage ≥ 80%
- [ ] All tests passing before Phase 3
