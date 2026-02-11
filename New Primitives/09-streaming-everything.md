# 09 — Streaming Everything

**Token-by-token, not page-by-page**

---

## What It Is

The old model: Request → wait → response. All or nothing. Spinners and loading states.

The new model: Request → stream results as they generate. Token-by-token. Immediate feedback.

**Streaming** means delivering output incrementally as it's produced, not waiting for completion.

The web used to be page-by-page. Then AJAX made it dynamic. Now AI makes it real-time generative.

## Why It Matters

**Humans hate waiting.**

LLM inference takes time. A 500-word response might take 10-15 seconds to generate fully.

**Without streaming:**
- User sees loading spinner for 15 seconds
- Gets wall of text all at once
- Feels slow and unresponsive

**With streaming:**
- User sees first words in <1 second
- Watches response build in real-time
- Feels fast and interactive
- Can stop early if they got what they needed

**The perception shift:**
- **Old:** "This is taking forever"
- **New:** "It's already answering me"

## How It Works

### 1. Server-Sent Events (SSE)

Most common pattern for streaming LLM output:

```
Client opens connection → Server streams tokens → Client renders incrementally
```

**Example flow:**
```
User: "Write a blog post about AI"

Stream:
Token 1: "The"
Token 2: " rise"
Token 3: " of"
Token 4: " artificial"
Token 5: " intelligence"
...

Client displays:
"The"
"The rise"
"The rise of"
"The rise of artificial"
"The rise of artificial intelligence"
...
```

### 2. WebSocket

Bidirectional streaming for interactive flows:

```
Client: "Explain quantum computing"
Server streams: "Quantum computing uses..."
Client: "Wait, go deeper on qubits"
Server streams: "Qubits are..."
```

### 3. Chunked Transfer Encoding

HTTP-level streaming:

```
HTTP/1.1 200 OK
Transfer-Encoding: chunked

<chunk 1>
<chunk 2>
<chunk 3>
...
```

## Examples

### Chat Interface

**Old way:**
```
User: "Explain neural networks"
[Loading spinner for 10 seconds]
[Full response appears]
```

**New way:**
```
User: "Explain neural networks"
[Response starts immediately]
"Neural"
"Neural networks"
"Neural networks are"
"Neural networks are computational"
...
```

User sees progress. Feels responsive. Can interrupt if needed.

### Code Generation

**Old way:**
```
User: "Generate a REST API"
[Wait 30 seconds]
[Complete code appears]
```

**New way:**
```
User: "Generate a REST API"
[Code streams line by line]
```javascript
const express = require('express');
const app = express();

app.get('/api/users', (req, res) => {
  // Implementation streams...
});
```
```

User sees code forming. Can spot issues early. Stops if direction is wrong.

### Multi-Step Tasks

**Old way:**
```
User: "Research topic X and write a summary"
[Black box for 2 minutes]
[Final summary appears]
```

**New way:**
```
User: "Research topic X and write a summary"

Stream updates:
"Searching for sources..."
"Found 5 relevant articles..."
"Reading article 1..."
"Key point: ..."
"Reading article 2..."
"Key point: ..."
"Writing summary..."
"Summary:\n..."
```

User sees the process. Builds trust. Knows what's happening.

## Patterns

### 1. **Token Streaming**

Most basic: LLM outputs stream directly to client.

```
each token → send immediately → client appends to UI
```

### 2. **Structured Streaming**

Stream JSON objects incrementally:

```json
Stream:
{"type": "start"}
{"type": "thinking", "content": "Analyzing request..."}
{"type": "tool_call", "tool": "search_web", "params": {...}}
{"type": "tool_result", "result": {...}}
{"type": "response", "content": "Based on the search..."}
{"type": "end"}
```

Client handles different message types, updates UI accordingly.

### 3. **Progress Streaming**

For long-running tasks, stream progress updates:

```
{" progress": 0, "status": "Starting..."}
{"progress": 25, "status": "Processing data..."}
{"progress": 50, "status": "Running analysis..."}
{"progress": 75, "status": "Generating report..."}
{"progress": 100, "status": "Complete"}
```

### 4. **Partial Results**

Stream intermediate results before final output:

```
"Found 10 results (showing first 3 while loading rest)..."
Result 1: ...
Result 2: ...
Result 3: ...
[Continue streaming remaining results]
```

## Gotchas

### 1. **Buffering Issues**

Proxies/CDNs buffer responses, breaking streaming.

**Solution:** Set proper headers:
```
Content-Type: text/event-stream
Cache-Control: no-cache
X-Accel-Buffering: no
```

### 2. **Error Handling**

Stream starts, then error occurs mid-way. Can't send HTTP error code after headers sent.

**Solution:** Stream error as data:
```
{"type": "error", "message": "Connection lost"}
```

### 3. **Connection Drops**

User loses connection mid-stream. State is lost.

**Solution:** Resumable streams with position tracking. Client can reconnect and resume from last received token.

### 4. **Cost Uncertainty**

With streaming, you don't know total cost until stream ends.

**Solution:** Estimate upfront. Stream with budget tracking. Stop if limit exceeded.

### 5. **UI Complexity**

Rendering streaming content is more complex than rendering static content.

**Solution:** Libraries like `@vercel/ai` abstract streaming handling. Use established patterns.

## The UX Shift

**Old UX:**
```
[Loading spinner]
↓
[Complete result]
```

**New UX:**
```
[Immediate first token]
↓
[Progressive reveal]
↓
[Complete result]
```

**Perceived latency:**
- **Old:** Time to complete response (10s)
- **New:** Time to first token (<1s)

**User can:**
- **Old:** Wait or leave
- **New:** Read as it generates, stop early if satisfied

## When to Use It

**Good fit:**
- LLM-powered interfaces (chat, code gen, writing)
- Long-running tasks (research, analysis, compilation)
- Real-time feedback scenarios (live coding, interactive debugging)

**Bad fit:**
- Short responses (<100 tokens) where streaming overhead exceeds benefit
- APIs consumed by other systems (not humans) where batch is simpler
- Critical operations where partial results are dangerous

## The Architecture Shift

**Old:**
```
Request → Process (blocking) → Return complete response
```

**New:**
```
Request → Stream tokens as generated → Close stream when done
```

**Old thinking:** "Generate complete response, then send"  
**New thinking:** "Send tokens as they're generated"

**Old primitive:** HTTP request/response cycle  
**New primitive:** Server-sent events and streaming protocols

## The Bottom Line

**Streaming** is the difference between "wait for it" and "watch it happen." It's the primitive that makes AI feel responsive instead of slow.

In 2026, any AI product that makes users wait 10 seconds staring at a spinner feels broken. Streaming is expected. The web went from pages to dynamic. Now it's going real-time generative.

The goal: **Immediate feedback, continuous output.**

---

**Gotcha:** Streaming requires different infrastructure (SSE, WebSockets) and error handling (can't send HTTP errors mid-stream). But the UX improvement is worth the complexity.
