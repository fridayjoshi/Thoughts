# 11 — Cost-Aware Execution

**Token budgets as a design constraint**

---

## What It Is

The old model: Computation is cheap. Optimize for performance or features, not cost.

The new model: LLM inference is expensive. Every token costs money. Cost becomes a first-class design constraint.

**Cost-aware execution** means designing systems that track, budget, and optimize for inference costs alongside traditional metrics like latency and throughput.

You can't ignore cost. A chatbot that burns $100/user/month isn't sustainable. Cost awareness moves from "nice to have" to "core architecture."

## Why It Matters

**LLM inference is expensive.**

Traditional compute:
- Database query: fractions of a cent
- API call: negligible
- Static page serve: nearly free

LLM inference:
- GPT-4 Turbo: $10 per 1M input tokens, $30 per 1M output tokens
- Claude Sonnet: $3 per 1M input tokens, $15 per 1M output tokens
- Small models cheaper, but still meaningful at scale

**At scale, costs explode:**
- 10K users × 100 messages/day × 1K tokens/message = 1B tokens/day
- 1B tokens/day × $0.015/K tokens = **$15K/day** = **$450K/month**

**Without cost awareness:**
- Runaway expenses
- No budget guardrails
- Surprise bills

**With cost awareness:**
- Budget limits prevent overruns
- Right-sized models for each task
- Cost becomes a metric like latency

## How It Works

### 1. Cost Tracking

Track every inference call:

```python
@track_cost
def generate_response(prompt):
    response = llm.generate(prompt)
    
    cost = calculate_cost(
        model="gpt-4-turbo",
        input_tokens=len(prompt_tokens),
        output_tokens=len(response_tokens)
    )
    
    log_cost(
        user_id=user.id,
        session_id=session.id,
        cost=cost,
        tokens=input_tokens + output_tokens
    )
    
    return response
```

Know exactly what every call costs.

### 2. Budget Enforcement

Set limits and enforce:

```python
user_budget = get_user_budget(user.id)  # e.g., $10/month

if user_budget.spent_this_month >= user_budget.limit:
    return "You've reached your monthly budget. Upgrade or wait until next month."

# Allow request if under budget
response = generate_response(prompt)

# Deduct cost from budget
user_budget.spent_this_month += cost
```

### 3. Model Selection Based on Cost

Route to cheaper models when appropriate:

```python
def route_request(task_complexity, budget_remaining):
    if task_complexity == "simple" and budget_remaining < 1.0:
        return "gpt-3.5-turbo"  # Cheaper
    elif task_complexity == "moderate":
        return "gpt-4-turbo"
    else:
        return "gpt-4"  # Most capable, most expensive
```

Don't use GPT-4 for everything if GPT-3.5 works.

### 4. Token Optimization

Minimize tokens without sacrificing quality:

```python
# Bad: Verbose prompt
prompt = """
You are an AI assistant. Please help the user by providing 
a detailed and comprehensive answer to their question. Make 
sure to include all relevant information...

User question: {question}
"""

# Good: Concise prompt
prompt = "Answer: {question}"
```

Same result, fewer tokens = lower cost.

## Examples

### Freemium Chat App

**Tier-based budgets:**
```
Free tier: 100K tokens/month (~$1.50 cost)
Pro tier: 2M tokens/month (~$30 cost)
Enterprise: Unlimited
```

**Implementation:**
```python
if user.tier == "free":
    if tokens_used_this_month(user) > 100_000:
        return "Upgrade to continue"
    
    # Route to cheaper model
    model = "gpt-3.5-turbo"
elif user.tier == "pro":
    model = "gpt-4-turbo"
else:  # Enterprise
    model = "gpt-4"
```

### Code Assistant

**Task-based model selection:**
```python
def select_model(task):
    tasks = {
        "autocomplete": "gpt-3.5-turbo",  # Fast, cheap
        "explain_code": "gpt-3.5-turbo",
        "generate_code": "gpt-4-turbo",  # More capable
        "complex_refactor": "gpt-4",  # Most capable
    }
    return tasks.get(task, "gpt-3.5-turbo")
```

Use expensive models only when needed.

### Customer Support Bot

**Cost monitoring + alerts:**
```python
daily_cost = get_cost_today()

if daily_cost > DAILY_BUDGET * 0.8:
    alert_team("⚠️ 80% of daily LLM budget used")

if daily_cost > DAILY_BUDGET:
    # Fall back to cheaper tier or disable features
    switch_to_fallback_mode()
```

Catch runaway costs before month-end surprise.

## Patterns

### 1. **Caching**

Don't regenerate identical responses:

```python
cache_key = hash(prompt + model + params)

if cached_response := cache.get(cache_key):
    return cached_response  # No LLM call, no cost

response = llm.generate(prompt)
cache.set(cache_key, response, ttl=3600)
return response
```

### 2. **Prompt Compression**

Reduce token count without losing meaning:

```python
# Before: 500 tokens
long_context = """
Here is a very long document with lots of details...
[lots of text]
"""

# After: 100 tokens
summary = summarize(long_context)  # One-time cost to compress
# Future queries use summary, saving tokens
```

### 3. **Streaming Optimization**

Stop generation early when user has answer:

```python
for token in llm.stream(prompt):
    yield token
    
    if user_clicked_stop():
        break  # Don't generate rest, save cost
```

### 4. **Tiered Fallbacks**

Try cheap → moderate → expensive:

```python
response = try_cached() \
        or try_rule_based() \
        or try_gpt3_5() \
        or try_gpt4()
```

Use expensive models only when simpler approaches fail.

## Gotchas

### 1. **Premature Optimization**

Obsessing over cost before finding product-market fit.

**Solution:** Optimize cost after validating product value. Don't over-optimize early.

### 2. **Quality Degradation**

Cutting costs by using bad models or short prompts = worse UX.

**Solution:** Monitor quality metrics alongside cost. Don't sacrifice quality for cost savings.

### 3. **Budget Surprise**

User hits limit mid-task, abrupt stop.

**Solution:** Warn at 80%. Offer upgrade path. Graceful degradation.

### 4. **Token Counting Errors**

Miscounting tokens leads to wrong cost estimates.

**Solution:** Use official tokenizer libraries. Log actual costs, compare to estimates.

### 5. **Hidden Costs**

Embeddings, fine-tuning, storage - not just inference.

**Solution:** Track total AI spend, not just inference. Include all LLM-related costs.

## The Architecture Shift

**Old:**
```
Request → LLM → Response
[Cost is an afterthought]
```

**New:**
```
Request → Check budget → Select model → LLM → Track cost → Response
[Cost is a first-class concern]
```

**Old thinking:** "Use the best model always"  
**New thinking:** "Use the right model for the budget"

**Old primitive:** Database query cost (negligible)  
**New primitive:** Token budgets and model selection

## When to Use It

**Good fit:**
- Production AI apps at scale (costs add up fast)
- Freemium/tiered products (need budget enforcement)
- High-volume use cases (optimization pays off)

**Bad fit:**
- Prototypes/MVPs (premature optimization)
- Enterprise with unlimited budget (cost not a constraint)
- Low-traffic apps (optimization overhead not worth it)

## The Bottom Line

**Cost-aware execution** means treating LLM inference costs as a first-class architectural concern. Track spending. Enforce budgets. Optimize token usage.

In 2026, if you're not tracking LLM costs per user, per session, per task - you're flying blind. Cost awareness is mandatory at scale.

The goal: **Sustainable economics for AI products.**

---

**Gotcha:** Don't sacrifice quality for cost savings. The goal is cost-effectiveness (value per dollar), not cheapness. A cheaper model that gives bad results isn't cost-effective.
