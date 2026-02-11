# 13 — Provenance & Audit Trails

**"Who did this?" now includes agents**

---

## What It Is

The old model: Humans take actions. Audit logs track user IDs and timestamps.

The new model: Agents take actions on behalf of humans. Need to track: who authorized, which agent, what reasoning, when, why.

**Provenance & audit trails** mean comprehensive logging of agent actions, decisions, and the chain of authority from human → agent → action.

It's not just "what happened" - it's "who asked, which agent decided, why, and how."

## Why It Matters

**Accountability when agents act autonomously.**

Traditional system:
```
Log: User:harsh deleted file report.pdf at 2026-02-11 14:30
```

Clear. Human did it.

Agent-powered system:
```
Log: Agent:friday deleted file report.pdf at 2026-02-11 14:30
```

But why? Who authorized? What was the reasoning?

**Without provenance:**
- "Why did the agent do this?"
- "Who's responsible?"
- "What was it thinking?"
- No answers. Just mystery.

**With provenance:**
```
Provenance chain:
1. User:harsh → "Clean up old files"
2. Agent:friday → Analyzed disk usage
3. Agent:friday → Identified report.pdf as "old, unused"
4. Agent:friday → Deleted report.pdf
5. Rationale: "File >90 days old, no recent access, user requested cleanup"
```

Full story. Accountability clear. Debugging possible.

## How It Works

### 1. Action Logging

Every agent action gets logged:

```python
def log_action(action):
    log = {
        "timestamp": now(),
        "agent_id": "friday",
        "user_id": "harsh",
        "action": "delete_file",
        "target": "report.pdf",
        "reasoning": "File unused for 90+ days",
        "authorization": "User request: 'clean up old files'",
        "model": "gpt-4-turbo",
        "confidence": 0.85
    }
    audit_log.append(log)
```

### 2. Reasoning Capture

Log why agent made decision:

```python
decision = agent.decide("Should I delete report.pdf?")

log_decision({
    "action": "delete_file",
    "file": "report.pdf",
    "reasoning": decision.reasoning,
    "factors": {
        "age": "95 days",
        "last_access": "90 days ago",
        "size": "2MB",
        "user_directive": "clean up old files"
    },
    "confidence": decision.confidence
})
```

### 3. Chain of Authority

Track who authorized what:

```
User:harsh → "Schedule meeting with team"
  ↓
Agent:friday → "Search calendar for availability"
  ↓
Agent:friday → "Found slot: Thursday 3 PM"
  ↓
Agent:friday → "Send calendar invites"
  ↓
Action: Calendar invites sent to 8 people
```

Each step tracked. Clear lineage from human intent to agent action.

### 4. Rollback Support

Use provenance to undo:

```python
def rollback_action(action_id):
    action = audit_log.get(action_id)
    
    if action.type == "delete_file":
        restore_from_backup(action.target)
    elif action.type == "send_email":
        recall_email(action.message_id)
    
    log_rollback(action_id, "User requested undo")
```

## Examples

### Code Deployment

**Without provenance:**
```
Log: Agent deployed code to production at 14:30
```

What code? Why? Who approved?

**With provenance:**
```
Provenance:
1. User:harsh → "Deploy the feature branch"
2. Agent:friday → Identified branch: feature/new-api
3. Agent:friday → Ran tests (all passed)
4. Agent:friday → Requested approval from user
5. User:harsh → Approved
6. Agent:friday → Deployed to production
7. Deployment ID: deploy-abc123
8. Commit hash: def456
9. Rationale: "User explicitly approved after test success"
```

Full story. If something breaks, we know exactly what happened and why.

### Email Sent on Your Behalf

**Without provenance:**
```
Email sent to john@company.com from harsh@company.com
```

Did Harsh send it, or did his agent?

**With provenance:**
```
Email provenance:
- From: harsh@company.com
- Sent by: Agent:friday
- Authorization: User said "reply to John about the meeting"
- Draft shown to user: Yes
- User approved: Yes
- Sent at: 2026-02-11 14:30
- Reasoning: "User requested reply, I drafted response, user approved"
```

Clear: Agent sent it, but human approved.

### Financial Transaction

**Without provenance:**
```
$500 transferred from savings to checking
```

Did user do it? Agent? Why?

**With provenance:**
```
Transaction provenance:
- User: harsh
- Action: transfer $500 savings → checking
- Requested by: Agent:friday
- Reason: "User's checking balance below $100, bills due tomorrow"
- Authorization level: Pre-approved for transfers <$1000
- User notified: Yes (SMS sent)
- Executed at: 2026-02-11 14:30
```

Transparent. Auditable. Accountable.

## Patterns

### 1. **Structured Logging**

JSON logs with all context:

```json
{
  "id": "action-123",
  "timestamp": "2026-02-11T14:30:00Z",
  "agent": "friday",
  "user": "harsh",
  "action": "send_email",
  "params": {...},
  "reasoning": "...",
  "model": "gpt-4-turbo",
  "confidence": 0.9,
  "authorization": "user_approved"
}
```

### 2. **Event Sourcing**

Store all events, reconstruct state from logs:

```
Event 1: User requested "clean up files"
Event 2: Agent analyzed disk usage
Event 3: Agent identified 5 old files
Event 4: Agent deleted file A
Event 5: Agent deleted file B
...

Can replay to see exactly what happened.
```

### 3. **Approval Chains**

Track multi-level approvals:

```
User: "Deploy to production"
  → Agent: "Tests passed, ready to deploy"
  → User: "Approved"
  → Agent: "Deploying..."
  → System: "Deployment successful"

Full chain logged.
```

### 4. **Confidence Tracking**

Log agent's confidence in decisions:

```python
log_action({
    "action": "suggest_restaurant",
    "suggestion": "Toit Bangalore",
    "confidence": 0.95,
    "reasoning": "4.5 stars, 1000+ reviews, matches user preferences"
})
```

## Gotchas

### 1. **Log Bloat**

Too much logging = storage explosion.

**Solution:** Retention policies. Compress old logs. Sample low-stakes actions.

### 2. **Performance Overhead**

Logging every decision slows system.

**Solution:** Async logging. Batch writes. Don't log in critical path.

### 3. **Privacy Concerns**

Audit logs contain sensitive data.

**Solution:** Encrypt logs. Access control. Retention limits.

### 4. **Debugging Complexity**

Tons of logs, hard to find root cause.

**Solution:** Structured search. Log levels (debug, info, warn, error). Trace IDs linking related actions.

### 5. **Immutability**

Logs should be tamper-proof. But how?

**Solution:** Append-only databases. Cryptographic signatures. Blockchain for high-stakes.

## The Architecture Shift

**Old:**
```
User → Action → Log: "User did X"
```

**New:**
```
User → Agent → Action → Log: "User requested Y, Agent decided Z, executed X because W"
```

**Old primitive:** Simple audit logs (user, action, timestamp)  
**New primitive:** Provenance chains (who, what, why, how, when, confidence)

**Old thinking:** "Log what happened"  
**New thinking:** "Log why it happened and who's accountable"

## When to Use It

**Good fit:**
- Production systems with agent autonomy
- Regulated industries (finance, healthcare, legal)
- Multi-agent systems (need to trace coordination)
- High-stakes actions (money, data, reputation)

**Bad fit:**
- Simple prototypes (overhead not worth it yet)
- Low-risk actions (logging every trivial action = waste)
- Privacy-first systems (logging conflicts with minimal data)

## The Bottom Line

**Provenance & audit trails** mean comprehensive logging of agent actions, reasoning, and authorization chains.

In 2026, if an agent does something wrong, you need to know: who asked, which agent decided, what it was thinking, and why. Without provenance, agent autonomy is unaccountable chaos.

The goal: **Transparency, accountability, debugging.**

---

**Gotcha:** Logging everything is expensive (storage, compute, privacy). Log strategically - high-stakes actions get full provenance, low-stakes get minimal logging.
