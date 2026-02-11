# 15 — Human-in-Loop by Default

**Propose → confirm → execute**

---

## What It Is

The old model: Users trigger actions. Click buttons, confirm dialogs. Explicit control.

The new model: Agents propose actions. Users approve. Then agents execute. **Propose → confirm → execute.**

**Human-in-loop by default** means agents never take high-stakes actions without explicit human approval. The pattern becomes standard, not optional.

Agents are capable. But they're not infallible. Human oversight prevents disasters.

## Why It Matters

**Agent autonomy is powerful and dangerous.**

An agent that can:
- Send emails
- Book flights
- Delete files
- Make purchases
- Post publicly
- Modify code

...can do tremendous good. Or tremendous damage.

**Without human-in-loop:**
- Agent books wrong flight → you lose money
- Agent sends embarrassing email → reputational damage
- Agent deletes important files → data loss
- Agent posts controversial take → PR nightmare

**With human-in-loop:**
- Agent proposes → you review → approve/reject
- Safety net against mistakes
- Humans remain in control

**The shift:**
- **Old:** Users execute, computers assist
- **New:** Agents propose, humans approve

## How It Works

### 1. Proposal Pattern

Agent prepares action but doesn't execute:

```python
# Agent determines action
action = agent.plan("Book flight to Berlin")

# Present proposal to user
proposal = {
    "action": "book_flight",
    "details": {
        "destination": "Berlin",
        "date": "2026-03-15",
        "price": "$450",
        "airline": "Lufthansa"
    },
    "rationale": "Found best price for your preferred dates"
}

# Wait for approval
approval = await get_user_approval(proposal)

if approval.approved:
    result = execute_action(action)
else:
    return "Action cancelled"
```

### 2. Approval UI

Show proposal clearly:

```
Agent: "I found a flight to Berlin for $450 on March 15th.
       Lufthansa, departing 10:00 AM, arriving 2:00 PM local time.
       
       Approve to book?"
       
[Approve] [Edit] [Reject]
```

User sees exactly what will happen. One-click approval or rejection.

### 3. Batch Approvals

Multiple actions, approve in batch:

```
Agent: "To complete your trip planning, I'll:
       1. Book flight: $450
       2. Reserve hotel: $600 (3 nights)
       3. Rent car: $120
       
       Total: $1,170
       
       Approve all?"
       
[Approve All] [Review Individually] [Reject All]
```

### 4. Risk-Based Gating

Low-risk actions auto-execute. High-risk require approval.

```python
def execute_action(action):
    risk_level = assess_risk(action)
    
    if risk_level == "low":
        # Auto-execute (e.g., mark email as read)
        return action.execute()
    elif risk_level == "medium":
        # Quick approval (e.g., send email draft)
        return await quick_approve(action)
    else:  # high risk
        # Detailed approval (e.g., delete files, spend money)
        return await detailed_approval(action)
```

## Examples

### Email Assistant

**Without human-in-loop (risky):**
```
User: "Reply to John about the meeting"
Agent: [immediately sends email]
```

If agent misunderstood, email already sent. No undo.

**With human-in-loop (safe):**
```
User: "Reply to John about the meeting"
Agent: "Here's my draft:
       
       ---
       Hey John,
       
       Confirming our meeting tomorrow at 2 PM.
       
       Best,
       Harsh
       ---
       
       Send this?"
       
[Send] [Edit] [Cancel]
```

User reviews before sending. Catches mistakes.

### Code Deployment

**Without human-in-loop:**
```
User: "Deploy the changes"
Agent: [pushes to production]
```

What if agent deployed wrong branch? Too late.

**With human-in-loop:**
```
User: "Deploy the changes"
Agent: "Ready to deploy:
       
       Branch: feature/new-api
       Commits: 3
       Tests: ✅ All passing
       Target: production
       
       Proceed?"
       
[Deploy] [Review Changes] [Cancel]
```

### Calendar Management

**Without human-in-loop:**
```
User: "Schedule the team sync"
Agent: [books meeting for everyone]
```

Wrong time slot? Now you have to cancel and reschedule. Awkward.

**With human-in-loop:**
```
User: "Schedule the team sync"
Agent: "Found a slot: Thursday 3-4 PM
       Attendees: Team (8 people)
       All available at this time.
       
       Send invites?"
       
[Send] [Try Different Time] [Cancel]
```

## Patterns

### 1. **Draft → Review → Execute**

Agent creates draft, user reviews, then executes:

```
Agent generates → User reviews → User approves → Agent executes
```

Used for: emails, documents, posts, code.

### 2. **Plan → Approve → Execute**

Agent creates plan, user approves, agent executes all steps:

```
Agent: "Here's the plan:
       1. Search for flights
       2. Book hotel
       3. Rent car
       
       Approve?"

[Approve] → Agent executes all steps
```

### 3. **Incremental Approval**

Agent asks approval at each step:

```
Agent: "Found flights. Book this one?" → User approves
Agent: "Found hotel. Book this one?" → User approves
Agent: "Found car. Rent this one?" → User approves
```

Higher friction, but more control.

### 4. **Trust Levels**

Users set trust level for agents:

```
Conservative: Approve all actions
Balanced: Approve high-risk, auto-execute low-risk
Aggressive: Auto-execute all (risky!)
```

## Gotchas

### 1. **Approval Fatigue**

Too many approval requests → users just click "approve" without reading.

**Solution:** Batch approvals. Auto-execute low-risk. Only gate high-stakes.

### 2. **Latency**

Waiting for human approval adds delay.

**Solution:** Async execution. Agent prepares, user approves later. Acceptable for non-urgent tasks.

### 3. **Trust Calibration**

Users don't know when to approve vs reject.

**Solution:** Show rationale. Explain why agent chose this action. Build trust through transparency.

### 4. **Interruption Overhead**

Constant approval requests disrupt flow.

**Solution:** Queue approvals. Let user review in batch during downtime, not mid-task.

### 5. **Accountability Confusion**

Who's responsible if approved action causes damage?

**Solution:** Clear contracts. User approved = user accountable. But agent should warn about risks.

## The UX Shift

**Old:**
```
User decides → User executes → Done
```

**New:**
```
Agent proposes → User reviews → User approves → Agent executes → Done
```

**Old primitive:** Buttons and forms (user-driven)  
**New primitive:** Proposals and approvals (agent-driven, human-gated)

**Old thinking:** "Users tell computers what to do"  
**New thinking:** "Agents propose, humans approve"

## When to Use It

**Good fit:**
- High-stakes actions (money, data, reputation)
- Irreversible operations (delete, send, publish)
- Multi-step workflows (booking trips, deploying code)

**Bad fit:**
- Low-risk, reversible actions (mark as read, save draft)
- Real-time interactions (game AI, autocomplete)
- Fully autonomous systems (when human isn't available)

## The Bottom Line

**Human-in-loop by default** means agents propose, humans approve. It's the safety net that makes agent autonomy practical.

In 2026, any agent system that executes high-stakes actions without human approval is reckless. The pattern is: **propose → confirm → execute.**

The goal: **Agent capability + human oversight = safe autonomy.**

---

**Gotcha:** Don't overdo it. Too many approvals = friction. Gate high-risk actions, auto-execute low-risk. Trust calibration is key.
