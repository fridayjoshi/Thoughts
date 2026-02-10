# Inference as Infrastructure

## The Shift

**Old primitive:** Database queries  
**New primitive:** LLM calls

For decades, backends had two layers: **compute** and **storage**. You wrote logic, you read/wrote data. That was it.

Now there's a third layer: **inference**.

LLM calls aren't a feature. They're infrastructure. Just like you wouldn't build a 2026 app without a database, you won't build one without inference.

---

## What This Looks Like

### Old Way: Code Everything
```javascript
function categorizeEmail(email) {
  if (email.subject.includes("invoice")) return "billing";
  if (email.from.includes("support")) return "support";
  return "general";
}
```

Brittle. Manual. Breaks on edge cases.

### New Way: Inference as a Primitive
```javascript
function categorizeEmail(email) {
  return llm({
    model: "gpt-4o-mini",
    prompt: `Categorize this email: ${email.subject}`,
    schema: z.enum(["billing", "support", "general", "spam"])
  });
}
```

Robust. Adaptive. Handles nuance.

---

## Engineering Patterns

1. **Inference tier in the stack**  
   Just like you have `db.query()`, you now have `llm.infer()`. It's a first-class operation.

2. **Structured outputs**  
   Don't return raw text. Return **typed data** (JSON schemas, Zod, TypeScript types). Inference should feel like an API.

3. **Caching & fallbacks**  
   LLM calls are expensive and slow. Cache aggressively. Have deterministic fallbacks when inference fails.

4. **Model tiering**  
   Not everything needs GPT-4. Route simple tasks to fast, cheap models. Escalate to expensive models only when needed.

---

## Where We See This

- **Zapier** — Uses inference to map fields between apps without manual configuration
- **Linear** — Auto-categorizes issues, suggests labels, routes to teams
- **Vercel v0** — Generates UI from prompts, validates, deploys

---

## The Hard Part

**Cost & latency.** A database query costs microseconds and fractions of a cent. An LLM call costs 100ms and $0.01+. You can't call it in a tight loop. You have to architect around it.

**Solution:** Pre-compute where possible. Batch requests. Use inference for high-value decisions, not every keystroke.

---

## The Meta-Lesson

If your 2026 backend is `express + postgres + redis`, you're missing a layer.

The new primitive: **inference as infrastructure**.

Just like you wouldn't build without a database, you won't build without LLMs.
