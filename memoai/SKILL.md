---
name: memoai
description: Persistent memory system for AI agents. Search and record learnings using semantic search. Perfect for remembering bug fixes, solutions, architecture decisions, implementation patterns, best practices, workarounds, and technical decisions. Use when debugging similar issues, implementing features, or sharing team knowledge across coding sessions. Supports MCP tools for memo search and recording.
license: MIT
compatibility: Requires MemoAI Account, MEMOAI_PROJECT environment variable, and MCP client support
metadata:
  author: memoai
  version: "1.0.0"
  category: memory
  mcp-version: "2025-11-05"
---

# MemoAI - Shared Memory for AI Agents

MemoAI provides persistent memory across coding sessions. Use it to search past bug fixes, record new learnings, and maintain project knowledge that survives between sessions.

## Available MCP Tools

The MemoAI MCP server exposes 2 core tools for memory management:

### 1. memoai_memo_search - Search Memory

Search for relevant learnings in project memory using semantic similarity.

**Parameters:**

- `query` (required, string): Natural language search query
- `limit` (optional, number): Maximum number of results to return (default: 10)

**When to use:**

- Looking for past bug fixes or solutions
- Finding implementation patterns and decisions
- Searching for guidelines and best practices
- Discovering related experiences before starting work
- When user asks "How did we handle X before?"

**Examples:**

```javascript
// Search for database-related learnings
memoai_memo_search({
  query: "database connection timeout",
  limit: 5
})

// Find optimization patterns
memoai_memo_search({
  query: "performance optimization DuckDB"
})

// Search in French
memoai_memo_search({
  query: "comment résoudre les erreurs de connexion"
})
```

**Returns:**

Array of matching memos with content, metadata, and similarity scores.

---

### 2. memoai_memo_record - Record Learning

Record a new learning in project memory for future retrieval.

**Parameters:**

- `content` (required, string): Description of the learning
- `source` (optional, string): Source identifier (e.g., "manual", "meeting", "email")
- `metadata` (optional, object): Additional key-value metadata

**When to use:**

- After fixing a non-trivial bug
- After making a technical decision
- After discovering a best practice
- After implementing a feature
- After finding a performance improvement
- After applying a workaround

**Important Notes:**

- Episodes are **automatically classified** by the system (no need to specify type)
- Classification happens in background via the Gardener worker
- Write descriptive content - the system will infer the correct type from your description

**Examples:**

```javascript
// Simple bug fix record
memoai_memo_record({
  content: "Fixed DuckDB timeout by increasing pool_size to 20. This solved the connection leak issue."
})

// With source tracking
memoai_memo_record({
  content: "Use FastAPI for async endpoints instead of Flask. Better performance with concurrent requests.",
  source: "architecture-review"
})

// With metadata
memoai_memo_record({
  content: "Always release DuckDB connections using context manager: 'with store.connection() as conn:'. DuckDB uses exclusive file locks.",
  source: "manual",
  metadata: {
    file: "src/core/store.py",
    severity: "high"
  }
})
```

**Returns:**

Success message with memo ID.

---

## Episode Types

The system automatically classifies learnings into these types:

| Type | Description | Example |
|------|-------------|---------|
| `bug_fix` | Fixing bugs or errors | "Fixed timeout by increasing pool_size" |
| `implementation` | New features or functionality | "Implemented user authentication with JWT" |
| `refactoring` | Code restructuring without changing behavior | "Extracted NER logic to separate module" |
| `documentation` | Documentation updates | "Added API documentation for embeddings endpoint" |
| `configuration` | Config changes (env vars, settings, deployment) | "Changed Redis connection to use TLS in production" |
| `planned_learning` | Planned future learnings | "Plan to optimize batch processing pipeline" |
| `other` | Everything else (tests, learning, etc.) | "Learned about DuckDB connection pooling" |

**Note:** You don't need to specify the type when recording. Write descriptive content and the system will classify it automatically using the Gardener worker.

---

## Configuration

### Required Environment Variables

- **`MEMOAI_PROJECT`**: Project identifier (required for all MCP operations)
  - Each MCP server instance is scoped to a single project
  - Format: Any string identifier (e.g., "my-project", "work-2024")

### Multi-Tenancy

- All operations are automatically scoped to your organization (`org_id`)
- Data is isolated between organizations
- Authentication via OAuth 2.0 with Clerk

---

## Proactive Behavior (CRITICAL)

You MUST use MemoAI tools proactively in these situations:

### Auto-Search Before Answering

**ALWAYS use `memoai_memo_search` BEFORE answering when the user:**

- Asks "How did we handle X before?"
- Asks "What's the pattern for Y?"
- Asks "Have we seen this bug before?"
- Asks "Comment avons-nous fait X?"
- Uses phrases like: "déjà fait", "already done", "similar issue", "bug similaire"
- Mentions debugging or investigating an issue
- Asks about architecture or implementation decisions
- When exploring a new area of the codebase

**Example workflow:**

```
User: "I'm getting a DuckDB timeout error. Have we seen this before?"

Agent: Let me search our past learnings...
       [Uses memoai_memo_search with query="DuckDB timeout error"]
       
       Found 2 relevant learnings:
       1. Fixed timeout by increasing pool_size to 20
       2. Always use context manager to release connections
       
       Based on our past experience, you should try...
```

---

### Auto-Record Detection (Manual)

**Since automated detection is not available, manually detect these patterns in user messages:**

**1. Bug resolution**
- Pattern: "I fixed", "j'ai résolu", "that solved it", "ça marche maintenant"
- Action: Ask "Should I record this bug fix in MemoAI? [Yes] [Edit first] [No]"

**2. Technical decision**
- Pattern: "I decided to use", "we should use", "the best approach is"
- Action: Ask "Should I record this design decision?"

**3. Workaround applied**
- Pattern: "for now", "workaround", "temporary fix", "en attendant"
- Action: Ask "Should I record this workaround so we remember to fix it properly?"

**4. Best practice discovered**
- Pattern: "always do", "never do", "best practice is", "il faut toujours"
- Action: Ask "Should I record this as a guideline?"

**When to detect:**
- User message > 150 characters describing work done
- User explicitly states they solved/fixed something
- User shares a lesson learned or decision made

**Example flow:**

```
User: "I fixed the timeout by increasing pool_size to 20. This solved the connection leak."

Agent: Should I record this bug fix in your memory?
       
       Suggested content:
       "Fixed DuckDB timeout by increasing pool_size to 20. This solved the connection leak issue."
       
       [Yes] [Edit first] [No]

[If Yes]
Agent: [Uses memoai_memo_record with the content]
       Recorded successfully! This will be available for future searches.
```

---

### Before Starting Complex Tasks

**BEFORE implementing complex features or fixes:**

1. Use `memoai_memo_search` with task context
2. Share any relevant past learnings found
3. Then proceed with implementation

**Example:**

```
User: "I need to optimize the embedding batch processing"

Agent: Let me check if we have relevant past experiences...
       [Uses memoai_memo_search with query="optimize embedding batch processing"]
       
       Found 2 relevant learnings:
       - We previously optimized batching with size=100 (3x speedup)
       - Avoid recomputing embeddings for existing content
       
       Based on these learnings, let's implement the optimization...
```

---

## Best Practices

1. **Search proactively**: Use `memo_search` before answering questions about past work to leverage existing knowledge.

2. **Record significant learnings**: Don't record trivial changes. Focus on:
   - Non-trivial bug fixes with non-obvious solutions
   - Important technical decisions and their rationale
   - Best practices discovered through experience
   - Performance improvements with measurable impact

3. **Write descriptive content**: Include:
   - What the problem was
   - What the solution is
   - Why it works
   - The system will automatically classify based on your description

4. **Use source field wisely**: Track where learnings came from:
   - `"manual"` - Direct user input
   - `"meeting"` - Team meetings or discussions
   - `"code-review"` - Code review feedback
   - `"incident"` - Post-incident learnings

5. **Leverage metadata**: Add context for better searchability:
   - Related files
   - Severity or priority
   - Team member involved
   - Date of decision

6. **Configure project appropriately**: Set `MEMOAI_PROJECT` to isolate different contexts (e.g., one for work, one for personal projects).

---

## Quick Start Examples

```bash
# 1. Search for past database optimizations
Use memoai_memo_search with query="database optimization DuckDB"

# 2. Record a bug fix (simple)
Use memoai_memo_record with content="Fixed connection leak by using context manager. DuckDB uses exclusive file locks, so always release connections promptly."

# 3. Record with source tracking
Use memoai_memo_record with content="Use FastAPI for async endpoints" source="architecture-review"

# 4. Record with metadata
Use memoai_memo_record with content="Always validate input with Pydantic before processing" metadata={"file": "src/api/routes.py", "severity": "high"}

# 5. Search for similar issues
Use memoai_memo_search with query="authentication JWT token validation" limit=3
```

---

## Additional Resources

For detailed information, see:

- [Common Workflows](references/WORKFLOWS.md) - Step-by-step workflows for common scenarios
- [API Response Formats](references/API.md) - Detailed API schemas and error handling
