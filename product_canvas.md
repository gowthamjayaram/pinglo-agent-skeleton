# Product Canvas Template — Agent Feature

Use this template to spec each agent feature for pinglo-agent-skeleton.

---

## 1. Headline

One sentence. What does this agent feature do? Who benefits?

---

## 2. Problem Statement

**The [User Persona]** struggles with [problem], because [reason] — [impact].

---

## 3. Target Users

**Persona:** Real users of Pinglo (e.g., User, Circle). NOT system components (Agent, Trigger, Tool).

| Persona | Impact |
|---|---|
| [User/Circle] | [Primary/Secondary] — [why they care] |

---

## 4. Strategic Positioning

- **Why now:** [What enables this feature to be built now?]
- **Strategic fit:** [How does this advance the agentic vision?]
- **Risk of skipping:** [What happens if we don't build this?]

---

## 5. User Stories & Acceptance Criteria

**IMPORTANT:** Acceptance criteria describe what the USER SEES/EXPERIENCES, NOT how the system implements it. ❌ Don't write: "agent calls Claude API". ✅ Do write: "user receives a message".

**Persona in Stories:** Use the persona from Section 3 (User, Circle, etc.). ❌ Never write "As an Agent", "As a Trigger", or "As a Tool". The agent/trigger are implementation details, not users.

### Story 1: [User Story Title]
> As a **[Persona]**, I want [what], so that [why].

**Acceptance criteria:**
- [ ] Given [user situation], when [user action], then [user outcome/experience]

---

## 6. Feature Scope

### In scope
- [What the agent does]
- [What inputs it accepts]
- [What outputs it returns]

### Out of scope
- [What the agent explicitly does NOT do]

### Assumptions & Technical Dependencies
- [What must already exist for this to work]
- [External dependencies (APIs, backend features, etc.)]

---

## 7. Success Metrics

- [How do we know this agent works?]
- [Test coverage expectations]
