# 10 — Statefulness Returns

**Sessions and multi-turn memory are back**

---

## What It Is

The old web: Stateless by design. Every request independent. REST principles everywhere.

The new AI era: Stateful by necessity. Conversations have context. Sessions persist across turns.

**Statefulness** means systems remember what happened before. Multi-turn conversations. Context that spans requests.

The pendulum swung from stateful (desktop apps, sessions) → stateless (REST, microservices) → stateful again (AI agents, conversations).

## Why It Matters

**Conversations aren't stateless.**

REST API thinking:
```
GET /api/users/123
POST /api/orders
DELETE /api/items/456
```

Each request is independent. No context. Works for CRUD.

**AI assistant thinking:**
```
User: "Find me flights to Berlin"
Agent: "When would you like to travel?"
User: "Next Tuesday"  ← Depends on previous turn
Agent: "Morning or evening?"
User: "Morning"  ← Depends on conversation history
```

Can't treat each message as independent. Need conversation state.

**The shift:**
- **Old:** Stateless requests (clean, scalable)
- **New:** Stateful sessions (necessary for coherence)

## How It Works

### 1. Session Management

Create and track conversation sessions:

```python
session = Session.create(user_id="harsh")
session.add_message("user", "Find flights to Berlin")
session.add_message("agent", "When would you like to travel?")
session.add_message("user", "Next Tuesday")

# Agent has full conversation history
response = agent.generate(session.get_history())
```

### 2. Context Window Management

LLMs have token limits. Can't include infinite history.

**Strategies:**
- **Sliding window:** Keep last N messages
- **Summarization:** Compress old messages
- **Semantic selection:** Keep most relevant messages

```python
if session.token_count() > MAX_TOKENS:
    # Summarize old messages
    summary = summarize(session.messages[:-10])
    session.messages = [summary] + session.messages[-10:]
```

### 3. State Persistence

Sessions survive server restarts:

```python
# Save session to database
session.save()

# Later, restore session
session = Session.load(session_id="abc123")
agent.resume(session)
```

### 4. Multi-Device State Sync

User switches devices mid-conversation:

```
Phone: "Find hotels in Berlin"
Agent: "Budget range?"
[User switches to laptop]
Laptop: "Under $150/night"  ← Continues same session
```

State syncs across devices.

## Examples

### Customer Support Bot

**Stateless (broken):**
```
User: "My order hasn't arrived"
Bot: "What's your order number?"
User: "ORDER-123"
Bot: "Found it. What's the issue?"
User: "It hasn't arrived"  ← Bot forgot we already said this
Bot: "Can you provide more details?"
```

**Stateful (works):**
```
User: "My order hasn't arrived"
Bot: "What's your order number?"
User: "ORDER-123"
Bot: [remembers issue] "I see ORDER-123 was shipped 3 days ago. 
     Let me check tracking..."
```

Bot remembers the original complaint.

### Code Assistant

**Stateless:**
```
User: "Write a React component"
Agent: [generates component]
User: "Add TypeScript types"
Agent: "Here's how to add TypeScript to React components..."
```

Doesn't remember the specific component. Generic answer.

**Stateful:**
```
User: "Write a React component"
Agent: [generates UserProfile component]
User: "Add TypeScript types"
Agent: [adds types to the UserProfile component it just wrote]
```

Remembers what it generated. Coherent edits.

### Travel Planning

**Stateless:**
```
User: "Book a flight to Tokyo"
Agent: "When?"
User: "March"
Agent: "Departing from?"
User: "San Francisco"
Agent: "Returning?"
User: "Two weeks later"
```

Every answer independent. Tedious Q&A.

**Stateful:**
```
User: "Book a flight to Tokyo"
Agent: "When would you like to go?"
User: "March"
Agent: "March works. Round trip from San Francisco?"
[Agent inferred location from user profile, remembered in session]
User: "Yes, two weeks"
Agent: [books SF → Tokyo, March, 2-week return]
```

Remembers answers. Infers from context. Natural flow.

## Patterns

### 1. **Pure Session State**

Everything in session object:

```python
session = {
    "user_id": "harsh",
    "messages": [...],
    "context": {
        "location": "Bangalore",
        "timezone": "Asia/Kolkata",
        "preferences": {...}
    }
}
```

### 2. **Hybrid State**

Session + external data:

```python
session.messages = [...]  # Conversation history
user_profile = db.get_user(session.user_id)  # Persistent user data
current_context = get_real_time_context()  # Live data

combined = session + user_profile + current_context
```

### 3. **Hierarchical Sessions**

Sessions within sessions:

```
Main session: "Plan my trip"
  ├─ Sub-session: "Book flights"
  ├─ Sub-session: "Find hotels"
  └─ Sub-session: "Plan activities"
```

Each sub-session has its own state, but inherits from parent.

### 4. **Ephemeral State**

State that expires:

```python
session.set("flight_search_results", results, ttl=300)  # 5 min
```

After timeout, state clears (not relevant anymore).

## Gotchas

### 1. **State Explosion**

Sessions grow huge. Every interaction adds state.

**Solution:** Prune old state. Summarize. Clear ephemeral data.

### 2. **Stale State**

User's context changed but session doesn't know.

Example: Session remembers "location: SF" but user moved to NYC.

**Solution:** Periodic state validation. Let users correct state manually.

### 3. **Concurrency Issues**

Same session, multiple devices, simultaneous interactions.

**Solution:** Last-write-wins, or operational transforms for conflict resolution.

### 4. **Privacy Concerns**

Sessions store sensitive conversation history.

**Solution:** Encrypt at rest. Let users delete sessions. Clear after inactivity.

### 5. **Memory Leaks**

Sessions never expire, accumulate indefinitely.

**Solution:** TTL on sessions. Auto-expire after inactivity.

## The Architecture Shift

**Old (stateless):**
```
Request → Handler → Response
[No memory between requests]
```

**New (stateful):**
```
Request → Session.load() → Handler(session) → Session.save() → Response
[Context persists across requests]
```

**Old primitive:** REST APIs, stateless microservices  
**New primitive:** Session stores, conversation management

**Old thinking:** "Make everything stateless for scalability"  
**New thinking:** "Statefulness is necessary for coherent AI interactions"

## When to Use It

**Good fit:**
- Conversational interfaces (chat, voice assistants)
- Multi-step workflows (booking, planning, troubleshooting)
- Collaborative sessions (shared context across users)

**Bad fit:**
- Simple CRUD operations (no conversation needed)
- High-scale read-only APIs (stateless scales better)
- One-shot requests (no benefit from session state)

## The Bottom Line

**Statefulness** means conversations have memory. Multi-turn context. Sessions that persist.

The web went stateless for scalability. But AI brought statefulness back - you can't have coherent conversations without remembering what came before.

In 2026, treating AI interactions as stateless requests is broken. Sessions and conversation history are primitives again.

The goal: **Coherent, context-aware interactions.**

---

**Gotcha:** Statefulness adds complexity (storage, sync, expiry). But for conversational AI, it's not optional - it's mandatory.
