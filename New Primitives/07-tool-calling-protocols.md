# 07 — Tool-Calling Protocols

**MCP as the new REST**

---

## What It Is

The old model: Apps expose REST APIs. Developers write code to integrate. Humans use UIs to interact.

The new model: Services expose tool-calling interfaces. Agents call tools directly. Humans describe what they want, agents orchestrate.

**Tool-calling protocols** define how agents discover, understand, and invoke external capabilities. The interface between AI and the world.

MCP (Model Context Protocol) is emerging as the standard. It's like REST for AI agents - a common language for tools.

## Why It Matters

**Agents need superpowers beyond language.**

An LLM alone can think and write. But to act, it needs tools:
- Read/write files
- Search the web
- Query databases
- Send emails
- Book flights
- Control hardware

**Without standardized protocols:**
- Every integration is custom
- Agents can't discover tools automatically
- No portability between systems

**With tool-calling protocols:**
- Tools expose capabilities in a standard format
- Agents discover and call tools dynamically
- One integration works across all agents

The shift: **From APIs for developers → to tool protocols for agents.**

## How It Works

### 1. Tool Definition

Services describe their capabilities in a standard format:

```json
{
  "name": "search_web",
  "description": "Search the web and return relevant results",
  "parameters": {
    "query": {
      "type": "string",
      "description": "Search query"
    },
    "max_results": {
      "type": "integer",
      "description": "Maximum number of results to return",
      "default": 10
    }
  }
}
```

### 2. Tool Discovery

Agent asks: "What tools are available?"

System responds with tool catalog:
```json
{
  "tools": [
    {
      "name": "search_web",
      "description": "Search the web...",
      ...
    },
    {
      "name": "send_email",
      "description": "Send an email message...",
      ...
    }
  ]
}
```

### 3. Tool Invocation

Agent decides: "I need to search the web"

Agent generates tool call:
```json
{
  "tool": "search_web",
  "parameters": {
    "query": "best restaurants in Bangalore",
    "max_results": 5
  }
}
```

System executes tool, returns result:
```json
{
  "results": [
    {"name": "Toit", "rating": 4.5, ...},
    {"name": "Smoke House Deli", "rating": 4.3, ...},
    ...
  ]
}
```

### 4. Result Integration

Agent receives result, continues reasoning:
```
User asked for restaurant recommendations.
I searched and found 5 options.
Top pick: Toit (4.5 rating, craft brewery, popular).
Let me format this into a helpful response...
```

## Examples

### File System Tools

**Old way:**
Developer writes code:
```python
import os
files = os.listdir('/path/to/dir')
```

**New way:**
Agent uses tool:
```json
{
  "tool": "filesystem_list",
  "parameters": {
    "path": "/path/to/dir"
  }
}
```

Agent decides when/how to use it based on user intent.

### Email Integration

**Old way:**
User clicks UI buttons, fills forms.

**New way:**
User: "Send the report to my team"

Agent:
1. Discovers `send_email` tool
2. Determines recipients from context
3. Formats report as attachment
4. Calls tool:
```json
{
  "tool": "send_email",
  "parameters": {
    "to": ["team@company.com"],
    "subject": "Weekly Report",
    "body": "...",
    "attachments": ["report.pdf"]
  }
}
```

No UI. No clicks. Just intent → action.

### Multi-Tool Orchestration

User: "Find the latest React docs and summarize them"

Agent orchestrates:
```json
Step 1:
{
  "tool": "search_web",
  "parameters": {
    "query": "React official documentation latest"
  }
}

Step 2:
{
  "tool": "fetch_url",
  "parameters": {
    "url": "https://react.dev/learn"
  }
}

Step 3:
{
  "tool": "summarize_text",
  "parameters": {
    "text": "<fetched content>",
    "max_length": 500
  }
}
```

Three tools, one flow, zero manual integration.

## MCP (Model Context Protocol)

Anthropic's proposed standard for tool-calling:

**Core concepts:**
1. **Servers** expose tools
2. **Clients** (agents) discover and call tools
3. **Protocol** defines communication format

**Benefits:**
- **Interoperability** — One MCP server works with all MCP clients
- **Composability** — Mix and match tools from different providers
- **Portability** — Agent built for MCP works everywhere

**Example MCP server:**
```
MCP Server: GitHub Integration

Tools:
- github_create_issue
- github_list_prs
- github_merge_pr
- github_get_file

Agent connects → discovers tools → calls as needed
```

## Patterns

### 1. **Single Tool Call**

Agent needs one capability:
```
User: "What's the weather?"
Agent → calls weather_api tool → returns result
```

### 2. **Sequential Chaining**

Agent calls tools in sequence:
```
User: "Book me a flight to Berlin"
Agent:
  1. search_flights
  2. check_budget
  3. book_flight
  4. send_confirmation_email
```

### 3. **Parallel Tool Execution**

Agent calls multiple tools simultaneously:
```
User: "Prepare for tomorrow's meeting"
Agent (parallel):
  - fetch_calendar_events
  - search_email_threads
  - get_attendee_profiles
  - summarize_last_meeting_notes
```

### 4. **Conditional Tool Use**

Agent decides which tool based on context:
```
User: "Find information about X"

If X is a person → use linkedin_search
If X is a place → use maps_api
If X is a topic → use web_search
```

## Gotchas

### 1. **Tool Hallucination**

Agent invents tools that don't exist or calls tools incorrectly.

**Solution:** Strict validation. Only allow defined tools. Return errors for invalid calls.

### 2. **Over-Calling**

Agent calls tools unnecessarily (expensive, slow).

**Solution:** Prompt engineering to encourage efficiency. Cost feedback loops.

### 3. **Security Risks**

Agent has access to powerful tools (delete files, send money). Misuse = damage.

**Solution:** Permission systems. Human-in-loop for high-risk actions. Audit trails.

### 4. **Discoverability Problems**

Too many tools = agent gets confused about which to use.

**Solution:** Clear, concise tool descriptions. Categorization. Context-aware tool filtering.

### 5. **Brittle Integrations**

Tool API changes break agent workflows.

**Solution:** Versioned protocols. Backward compatibility. Graceful degradation.

## The Architecture Shift

**Old:**
```
User → UI → REST API → Database
```

**New:**
```
User → Agent → Tool Protocol → Services
```

**Old primitive:** REST endpoints with JSON payloads  
**New primitive:** Tool definitions with semantic descriptions

**Old thinking:** "Build an API for developers"  
**New thinking:** "Build tools for agents"

**Old integration:** Developer writes glue code  
**New integration:** Agent discovers tools dynamically

## When to Use It

**Good fit:**
- Systems designed for agent interaction (not just humans)
- Multi-capability products (email + calendar + files + ...)
- Platforms where extensibility matters (users add custom tools)

**Bad fit:**
- Pure REST APIs for human developers (still has its place)
- Real-time systems where discovery overhead is too high
- Security-critical systems where dynamic tool calling is too risky

## The Bottom Line

**Tool-calling protocols** are the interface between agents and the world. They define how agents discover capabilities and invoke actions.

MCP is emerging as the standard - the REST of the AI era. Instead of developers writing integration code, agents discover and call tools dynamically.

The goal: **Agents with superpowers, via standardized tool interfaces.**

---

**Gotcha:** Tool-calling is powerful but risky. An agent with file system access can delete everything. Permission systems and audit trails are mandatory, not optional.
