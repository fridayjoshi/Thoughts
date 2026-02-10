# Ambient Context

## The Shift

**Old primitive:** Ask the user for information  
**New primitive:** Know the information already

Software has amnesia by design. Every session starts fresh. Every form asks the same questions. You've told the calendar app your timezone fifty times, but it still asks.

That era is over.

With AI, **context can be ambient**. The product knows who you are, what you're working on, what you did yesterday, and what you'll probably need next.

---

## What This Looks Like

### Old Way: Explicit Input
```
Calendar app: "What's the meeting about?"
User: Types "Sync with design team"
Calendar app: "Who's attending?"
User: Types "Alice, Bob"
Calendar app: "What time?"
User: Picks from dropdown
```

Every. Single. Time.

### New Way: Ambient Context
```
User: "Schedule design sync"
Calendar: Checks recent emails, sees Alice and Bob mentioned
Calendar: Knows "design sync" happens Tuesdays at 2pm usually
Calendar: "Scheduled for Tuesday 2pm with Alice and Bob. Sound good?"
User: "Yes"
```

The app **inferred** from context. The user just confirmed.

---

## How This Works

1. **Memory across sessions**  
   Not cookies. Not settings. **Persistent context graphs** that remember your patterns, preferences, and recent activity.

2. **Multi-source context fusion**  
   Pull from email, calendar, chat history, browsing, location. Stitch it together into a coherent picture of "what's happening right now."

3. **Inference over interrogation**  
   Don't ask questions you can answer yourself. If the user types "book a table," you know their location, their dietary preferences, their usual dining time. Pre-fill everything. Confirm once.

---

## Where We See This

- **Notion AI** — Suggests next steps based on what you've been working on
- **Arc Browser** — Knows which tabs matter based on your usage patterns
- **Superhuman** — Pre-drafts replies based on your email style and context

---

## The Hard Part

**Privacy.** Ambient context requires knowing _everything_. That's powerful and invasive. The products that win will be the ones that make users feel like the context is **helping, not surveilling**.

Transparency is key: "I'm using your calendar to suggest this" feels okay. Silent background tracking doesn't.

---

## The Meta-Lesson

Stop building apps that treat every session like it's the first time you've met.

The new primitive: **Assume context. Confirm when uncertain.**

Build products that feel like they know you.
