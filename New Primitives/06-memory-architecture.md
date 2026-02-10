# Memory Architecture

## The Shift

**Old primitive:** Sessions and cookies  
**New primitive:** Persistent context graphs

The web was built stateless. HTTP has no memory. Every request is independent. We hacked around it with cookies, sessions, local storage — but fundamentally, the model was **amnesiac by default**.

AI products can't work that way. Agents need memory. Not just "what did you click last session," but **semantic, long-term, queryable memory**.

This isn't new tech (vector databases existed before). What's new is that memory is now a **first-class primitive**, not a bolt-on.

---

## What This Looks Like

### Old Way: Session State
```javascript
// User logs in
session.user = { id: 123, name: "Alice" }

// User does stuff
session.cart = [item1, item2]

// Session expires
// Everything is gone
```

Shallow. Ephemeral. No continuity across sessions.

### New Way: Memory as a Graph
```javascript
// User mentions a project
memory.store({
  type: "project",
  name: "Redesign dashboard",
  relatedTo: ["Alice", "Bob", "Q1 goals"],
  timestamp: now()
});

// Three weeks later, user asks: "How's that dashboard thing going?"
memory.search("dashboard project") // Retrieves context
// Agent: "You mean the Q1 dashboard redesign with Alice and Bob?"
```

Semantic. Persistent. Contextual.

---

## Technical Components

1. **Vector stores (Pinecone, Weaviate, pgvector)**  
   Store embeddings of user activity, conversations, documents. Query semantically.

2. **RAG (Retrieval-Augmented Generation)**  
   Don't shove everything into the prompt. Retrieve relevant context dynamically.

3. **Hybrid search**  
   Combine semantic search (embeddings) with keyword search (BM25). Best of both worlds.

4. **Memory hierarchies**  
   - **Working memory:** Current session context (cheap, fast)
   - **Short-term memory:** Recent activity (days/weeks)
   - **Long-term memory:** Curated, compressed knowledge (months/years)

---

## Where We See This

- **Mem.ai** — Personal memory layer that works across all apps
- **Rewind.ai** — Records everything, makes it searchable
- **ChatGPT Memory** — Remembers preferences, facts, context across conversations

---

## The Hard Part

**What to remember vs. what to forget.** You can't store everything (cost, noise). But you can't forget everything (no continuity).

**Solution:** Active forgetting. Compress old memories. Summarize. Keep the essence, discard the noise.

---

## The Meta-Lesson

If your 2026 product forgets what the user did last week, you're building for 2015.

The new primitive: **memory as a queryable, semantic, persistent layer**.

Not sessions. Not cookies. **Memory**.
