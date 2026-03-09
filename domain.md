# Pinglo Agent Skeleton Architecture

## What is an Agent?

Agent = Program that uses Claude API to reason about situations, decide whether to take action, and call tools to execute decisions.

**Flow:**
1. Agent receives context (situation description)
2. Agent sends context + available tools to Claude
3. Claude reads context, reasons, decides if action needed
4. Claude calls tool (if needed) or finishes (if not)
5. Agent captures result, loops back if Claude wants more tools

---

## Agent Loop (Core)

```
Input Context
    ↓
Send to Claude API (with tools)
    ↓
Claude decides: "Use tool?" or "Done?"
    ├─ If tool needed: Extract tool call
    │  └─ Execute tool → Send result back → Loop
    └─ If done: Return final response
```

**Why this matters:** Agent isn't just following rules. Claude *reasons* about context, interprets ambiguity, and decides what to do.

---

## Use Cases

### Use Case 1: Inactivity Re-engagement

**Input:** User hasn't messaged Pinglo in > 7 days

**Agent reasoning:**
- "User is inactive"
- "I should send a friendly message"
- "Use receive_message tool"

**Tool call:** `receive_message(user_id="user-123", message="Hi! What's up, missing you")`

**Output:** Message sent to user via Pinglo

---

### Use Case 2: Activity Reminder

**Input:** Activity created, user tagged but didn't respond

**Example:** Ankita posts "Badminton Monday", tags @Gowtham. Gowtham doesn't reply.

**Agent reasoning:**
- "Gowtham was tagged in badminton activity"
- "Gowtham hasn't responded"
- "I should remind him with context"
- "Use receive_message tool"

**Tool call:** `receive_message(user_id="gowtham", message="Dude, Ankita invited you to badminton on Monday. Interested?")`

**Output:** Message sent to Gowtham via Pinglo

---

## Integration with Pinglo MCP

```
Agent
  ├─ Defines tool schema (receive_message)
  ├─ Sends context + schema to Claude
  ├─ Claude decides to call tool
  └─ Agent executes via HTTP:
      POST http://localhost:8000/messages/
      {
        "message": "...",
        "user_id": "..."
      }
```

**What agent knows:** URL, tool schema, expected response format

**What agent doesn't know:** How Pinglo backend works, NLU logic, database structure

---

## Constraints & Assumptions

**Exists:**
- Pinglo FastAPI backend (running)
- Pinglo MCP Server (running on localhost:8000)
- Claude API (accessible with API key)

**Doesn't exist (agent doesn't build):**
- DB queries (triggers handle this separately)
- Pinglo business logic (MCP Server handles)
- Message parsing/validation (MCP Server handles)

---

## What We're NOT Building

- Agent doesn't query Pinglo DB directly
- Agent doesn't implement NLU parsing
- Agent doesn't manage user sessions
- Agent doesn't validate messages

**Why:** Separation of concerns. Triggers handle "what to do", agent handles "how to decide".

---

## Key Design Principle

**Agent as Decision Maker, Not Executor**

Agent specializes in:
- Understanding ambiguous context
- Reasoning about intent
- Calling right tool for situation

Agent delegates to:
- Triggers (detect situations)
- Pinglo MCP (execute actions)
- Pinglo Backend (store/process data)
