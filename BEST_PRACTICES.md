# Pinglo Agent Skeleton Best Practices

## 1. Agent Loop Pattern

**Do:**
```python
def run_agent(context: str) -> dict:
    messages = [{"role": "user", "content": context}]

    while True:
        response = client.messages.create(
            model="claude-3-5-haiku-20241022",
            max_tokens=1024,
            tools=tools,
            messages=messages
        )

        if response.stop_reason == "tool_use":
            # Execute tool, append result, continue loop
            for block in response.content:
                if block.type == "tool_use":
                    result = execute_tool(block.name, block.input)
                    messages.append({"role": "assistant", "content": response.content})
                    messages.append({
                        "role": "user",
                        "content": [{"type": "tool_result", "tool_use_id": block.id, "content": str(result)}]
                    })
        else:
            # Claude finished
            break

    return response
```

**Don't:**
- Hardcode response expectations
- Skip error handling in tool execution
- Forget to append tool results back to messages
- Assume Claude always uses a tool

---

## 2. Error Handling

**Always wrap tool execution:**
```python
try:
    result = requests.post(
        url,
        json=tool_input,
        timeout=5
    )
    result.raise_for_status()
    return result.json()
except requests.Timeout:
    return {"success": False, "error": "Timeout"}
except requests.HTTPError as e:
    return {"success": False, "error": f"HTTP {e.response.status_code}"}
except Exception as e:
    return {"success": False, "error": str(e)}
```

**Key rule:** Return structured response, never raise exceptions to Claude.

---

## 3. Claude API Configuration

**Defaults:**
- `model`: "claude-3-5-haiku-20241022" (fast, cost-effective for skeleton)
  - Upgrade to "claude-opus-4-6" for complex reasoning tasks
- `max_tokens`: 1024 (sufficient for tool decisions)
- `temperature`: Not specified (defaults to 1.0, good for reasoning)
- `system` prompt: Optional (only if agent needs special behavior)

**Don't use:**
- temperature=0 (removes reasoning ability)
- "claude-3-5-sonnet" (overkill for simple tool decisions)

---

## 4. Configuration Management

**Use environment variables:**
```python
# config.py
PINGLO_MCP_SERVER_URL = os.getenv("PINGLO_MCP_SERVER_URL", "http://localhost:8000")
CLAUDE_API_KEY = os.getenv("CLAUDE_API_KEY")
```

**Never hardcode:**
- API keys
- Server URLs
- Timeouts
- Model names (make configurable)

---

## 5. Testing Agents

**Mock Claude and tools:**
```python
from unittest.mock import Mock, patch

# Mock tool execution
def mock_tool_call(name, input_data):
    return {"success": True, "data": {...}}

# Mock Claude response
mock_response = Mock()
mock_response.stop_reason = "tool_use"
mock_response.content = [
    Mock(type="tool_use", name="receive_message", id="call-123", input={"message": "...", "user_id": "..."})
]

with patch("anthropic.Anthropic.messages.create", return_value=mock_response):
    result = run_agent("test context")
```

**Test:**
- Agent uses correct tool
- Agent handles tool failures
- Agent terminates (doesn't loop forever)
- Agent appends tool results correctly

---

## 6. Logging

**Log at key points:**
```python
import logging
logger = logging.getLogger(__name__)

logger.info(f"Agent triggered with context: {context[:100]}...")
logger.debug(f"Claude response: {response.stop_reason}")
logger.info(f"Tool call: {tool_name}({list(tool_input.keys())})")
logger.info(f"Tool result: {result['success']}")
```

**Never log:**
- API keys
- Full user messages (PII)
- Claude API keys in responses

---

## Anti-Patterns

❌ Calling Claude multiple times for same context (waste, cost)
❌ Not appending tool results to messages (breaks loop, Claude loops)
❌ Assuming Claude always uses a tool (it might decide "no action needed")
❌ Catching exceptions and swallowing them (hidden bugs)
❌ Hardcoding configuration (brittleness, fails in different environments)
❌ Using complex system prompts (keep agent simple, let triggers handle context)
❌ Forgetting tool_use_id when returning tool results (Claude gets confused)
