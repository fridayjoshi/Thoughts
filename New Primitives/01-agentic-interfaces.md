# Agentic Interfaces

## The Shift

**Old primitive:** Design user flows  
**New primitive:** Design agent handoffs

For twenty years, we designed for humans clicking buttons. Every product was a graph of screens, forms, and actions. The user was the executor.

Now, the user is the **intent-giver**. Agents are the executors.

This isn't about adding a chatbot to your app. It's about recognizing that the **interaction model has fundamentally changed**.

---

## What This Looks Like

### Old Way: Linear User Flow
```
User → Search for flights → Filter results → Pick seat → Enter payment → Confirm
```

Every step is manual. Every decision point is a form.

### New Way: Agent Handoffs
```
User: "Book me a flight to Berlin next week, aisle seat, under $500"
  ↓
Calendar Agent: Checks schedule, finds open window
  ↓
Travel Agent: Searches flights, narrows options
  ↓
Payment Agent: Pre-fills payment, asks confirmation
  ↓
User: Approves
  ↓
Travel Agent: Books, sends confirmation
```

The user stated **intent**. Agents did the work. The interface was a conversation with checkpoints, not a sequence of screens.

---

## Design Implications

1. **Stop designing screens. Start designing handoff points.**  
   Where does the human hand control to an agent? Where does the agent hand back for approval?

2. **Implicit > Explicit**  
   Don't ask the user for information you can infer. Calendar knows their schedule. Payment agent knows their card.

3. **Conversation is the UI**  
   The primary interface isn't buttons — it's natural language with structured fallbacks.

4. **Multimodal by default**  
   Text, voice, images, actions — agents should handle all modalities fluidly.

---

## Where We See This

- **Perplexity** — Not search results, but researched answers with citation handoffs
- **Replit Agent** — Not a code editor, but "build me X" → agent codes, tests, deploys
- **OpenAI Canvas** — Not a doc editor, but a collaborative writing surface where the agent is a co-author

---

## The Hard Part

**Designing for ambiguity.** User flows are deterministic. Agent handoffs are probabilistic. You can't wireframe "the AI understood what I meant." You have to design for misunderstanding, confirmation loops, and progressive clarification.

---

## The Meta-Lesson

If your 2026 product still thinks in "pages" and "forms," you're building for 2015.

The new primitive is: **human states intent → agents execute → human confirms → loop**.

Design for that.
