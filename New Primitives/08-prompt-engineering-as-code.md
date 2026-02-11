# 08 — Prompt Engineering as Code

**Version, test, deploy prompts like APIs**

---

## What It Is

The old model: Prompts are copy-pasted strings in code. Hard to maintain, test, or version.

The new model: Prompts are first-class code artifacts. Versioned, tested, deployed like APIs.

**Prompt engineering as code** means treating prompts with the same rigor as production code - version control, testing, CI/CD, monitoring.

Prompts aren't throw-away strings. They're core product logic.

## Why It Matters

**Prompts are the new business logic.**

In traditional apps:
- Code implements behavior
- Tests ensure correctness
- Version control tracks changes
- Deployment pipelines push updates

In AI apps:
- **Prompts implement behavior**
- But they're often treated as afterthoughts
- Stored as strings in code
- Changed without testing
- No versioning or rollback

**The problem:**
- Prompt changes break things
- No audit trail of what changed when
- Hard to A/B test different prompts
- Debugging is painful (which prompt version caused this?)

**Prompt engineering as code fixes this:**
- Version control prompts like code
- Test prompts against expected outputs
- Deploy prompts through pipelines
- Monitor prompt performance in production

## How It Works

### 1. Prompt as Files

Store prompts in dedicated files, not hardcoded strings:

```
prompts/
  system-instructions.md
  code-review.md
  email-reply.md
  summarization.md
```

**Before:**
```python
response = llm.generate(
  "You are a helpful assistant. Be concise and direct. "
  "The user asked: {user_message}"
)
```

**After:**
```python
prompt = load_prompt("system-instructions.md")
response = llm.generate(prompt.format(user_message=msg))
```

### 2. Version Control

Track prompt changes in git:

```bash
$ git log prompts/code-review.md

commit abc123
Date: 2026-02-10
  Improve code review prompt - focus on security

commit def456
Date: 2026-02-09
  Initial code review prompt
```

Can roll back bad changes. See who changed what when.

### 3. Testing Prompts

Write tests for prompt outputs:

```python
def test_summarization_prompt():
    prompt = load_prompt("summarization.md")
    input_text = "Long article about AI..."
    
    output = llm.generate(prompt.format(text=input_text))
    
    assert len(output) < 500  # Summary should be short
    assert "AI" in output  # Should mention key topic
    assert is_coherent(output)  # Should make sense
```

Catch regressions. Ensure prompt changes don't break behavior.

### 4. Prompt Versioning

Multiple versions, deployed independently:

```
prompts/
  code-review-v1.md
  code-review-v2.md
  code-review-v3.md (current)
```

```python
# Production uses v3
prompt = load_prompt("code-review-v3.md")

# Canary testing v4 on 5% of traffic
if random() < 0.05:
    prompt = load_prompt("code-review-v4.md")
```

### 5. Deployment Pipelines

Prompts go through CI/CD:

```yaml
# .github/workflows/prompts.yml
on:
  push:
    paths:
      - 'prompts/**'

jobs:
  test-prompts:
    runs-on: ubuntu-latest
    steps:
      - name: Run prompt tests
        run: pytest tests/prompts/
      
      - name: Deploy to staging
        run: ./deploy-prompts staging
      
      - name: Deploy to production (manual approval)
        run: ./deploy-prompts production
```

Prompt changes reviewed, tested, deployed with oversight.

## Examples

### Code Assistant Prompts

**Structure:**
```
prompts/
  code-generation/
    system.md
    typescript.md
    python.md
    go.md
  code-review/
    system.md
    security-focused.md
    performance-focused.md
```

**Usage:**
```python
system_prompt = load_prompt("code-generation/system.md")
lang_prompt = load_prompt(f"code-generation/{language}.md")

prompt = f"{system_prompt}\n\n{lang_prompt}\n\nTask: {task}"
code = llm.generate(prompt)
```

### Email Assistant Prompts

**Versioning:**
```
prompts/email/
  reply-v1.md  # Original
  reply-v2.md  # Added tone detection
  reply-v3.md  # Current: improved context handling
```

**A/B Testing:**
```python
if user_id % 2 == 0:
    prompt = load_prompt("email/reply-v3.md")  # Current
else:
    prompt = load_prompt("email/reply-v4.md")  # Testing new version
```

Track metrics: which version gets better user ratings?

### Summarization Prompts

**Testing:**
```python
def test_summarization_preserves_key_points():
    test_cases = [
        {
            "input": "Article about climate change...",
            "expected_keywords": ["climate", "emissions", "temperature"]
        },
        {
            "input": "Article about AI safety...",
            "expected_keywords": ["AI", "safety", "alignment"]
        }
    ]
    
    prompt = load_prompt("summarization.md")
    
    for case in test_cases:
        summary = llm.generate(prompt.format(text=case["input"]))
        for keyword in case["expected_keywords"]:
            assert keyword.lower() in summary.lower()
```

## Patterns

### 1. **Template Variables**

Prompts with placeholders:

```markdown
# system-instructions.md

You are {assistant_name}, an AI assistant.

User preferences:
- Tone: {tone}
- Verbosity: {verbosity}
- Expertise level: {expertise}

Current context: {context}

User message: {user_message}
```

```python
prompt = load_prompt("system-instructions.md").format(
    assistant_name="Friday",
    tone="direct",
    verbosity="concise",
    expertise="high",
    context=get_context(),
    user_message=msg
)
```

### 2. **Layered Prompts**

Compose prompts from reusable pieces:

```python
base = load_prompt("base-instructions.md")
task = load_prompt("tasks/code-review.md")
constraints = load_prompt("constraints/security.md")

final_prompt = f"{base}\n\n{task}\n\n{constraints}"
```

### 3. **Environment-Specific Prompts**

Different prompts for dev/staging/production:

```python
env = get_environment()  # "dev", "staging", "prod"
prompt = load_prompt(f"prompts/{env}/system.md")
```

Allows testing aggressive prompts in dev without risking production.

### 4. **Conditional Prompt Selection**

Choose prompt based on context:

```python
if user.is_expert:
    prompt = load_prompt("expert-mode.md")
elif user.is_beginner:
    prompt = load_prompt("beginner-mode.md")
else:
    prompt = load_prompt("default.md")
```

## Gotchas

### 1. **Prompt Drift**

Production prompt diverges from version-controlled one (someone manually edits in prod).

**Solution:** Deploy from git. Make prod read-only.

### 2. **Test Brittleness**

LLM outputs are non-deterministic. Tests fail randomly.

**Solution:** Test properties, not exact outputs. Use fuzzy matching. Set temperature=0 for testing.

### 3. **Prompt Bloat**

Prompts grow huge over time as edge cases accumulate.

**Solution:** Regular refactoring. Remove outdated instructions. Keep prompts focused.

### 4. **Context Window Limits**

Long prompts + user messages hit token limits.

**Solution:** Tiered prompts. Load full prompt for simple queries, minimal prompt for complex ones.

### 5. **Debugging Difficulty**

Something breaks. Which prompt version? What was the input?

**Solution:** Log prompt version + inputs + outputs. Trace every generation.

## The Engineering Shift

**Old (prompts as strings):**
```python
response = llm.generate(
    "You are a helpful assistant..."
)
```

**New (prompts as code):**
```python
prompt = PromptTemplate.load("assistant-v3")
response = llm.generate(
    prompt.render(
        user=user,
        context=context
    )
)
```

**Old thinking:** "Prompts are just text"  
**New thinking:** "Prompts are product logic"

**Old primitive:** String interpolation  
**New primitive:** Prompt templates, versioning, testing

## When to Use It

**Good fit:**
- Production AI applications (prompts are critical)
- Multi-prompt systems (many prompts to manage)
- Teams (collaboration, review, accountability)

**Bad fit:**
- Prototypes/experiments (overhead not worth it yet)
- Single-prompt apps (file-based overkill for one prompt)
- Rapidly iterating (formal versioning slows down exploration)

## The Bottom Line

**Prompt engineering as code** means treating prompts with the same discipline as production code. Version control, testing, deployment pipelines, monitoring.

In 2026, if your prompts are hardcoded strings scattered across your codebase, you're doing it wrong. Prompts are core product logic. Treat them accordingly.

The goal: **Reliable, maintainable, testable prompt systems.**

---

**Gotcha:** Prompt engineering as code adds process overhead. Worth it in production, overkill for early exploration. Start simple, add rigor as you scale.
