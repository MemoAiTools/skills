# API Response Formats

Detailed API schemas and response formats for MemoAI MCP tools.

## Table of Contents

1. [memoai_memo_search](#memoai_memo_search)
2. [memoai_memo_record](#memoai_memo_record)
3. [Error Responses](#error-responses)
4. [Internal API Details](#internal-api-details)

---

## memoai_memo_search

Search through your memos using semantic search.

### Request Schema

```typescript
{
  query: string;      // Required: Search query
  limit?: number;     // Optional: Max results (default: 10, range: 1-100)
}
```

### Response Schema

```typescript
{
  content: [
    {
      type: "text",
      text: string  // JSON stringified array of search results
    }
  ]
}
```

### Search Result Object

Each result in the array contains:

```typescript
{
  id: string;              // UUID of the memo/episode
  content: string;         // The memo content
  episode_type: string;    // One of: bug_fix, implementation, refactoring, etc.
  created_at: string;      // ISO 8601 timestamp
  source?: string;         // Source identifier (e.g., "manual", "meeting")
  similarity?: number;     // Similarity score (0-1, higher is better)
  tags?: string[];         // Array of tags
  extra_metadata?: object; // Additional metadata
}
```

### Example Request

```javascript
memoai_memo_search({
  query: "DuckDB timeout error",
  limit: 5
})
```

### Example Response

```json
{
  "content": [
    {
      "type": "text",
      "text": "[\n  {\n    \"id\": \"123e4567-e89b-12d3-a456-426614174000\",\n    \"content\": \"Fixed DuckDB timeout by increasing pool_size to 20. This solved the connection leak issue.\",\n    \"episode_type\": \"bug_fix\",\n    \"created_at\": \"2024-01-15T10:30:00Z\",\n    \"source\": \"manual\",\n    \"similarity\": 0.89,\n    \"tags\": [],\n    \"extra_metadata\": {\n      \"file\": \"src/core/store.py\"\n    }\n  },\n  {\n    \"id\": \"223e4567-e89b-12d3-a456-426614174001\",\n    \"content\": \"Always release DuckDB connections using context manager. DuckDB uses exclusive file locks.\",\n    \"episode_type\": \"guideline\",\n    \"created_at\": \"2024-01-14T15:20:00Z\",\n    \"source\": \"manual\",\n    \"similarity\": 0.76,\n    \"tags\": [\"best-practice\"],\n    \"extra_metadata\": {}\n  }\n]"
    }
  ]
}
```

### Response Details

**Similarity Scoring:**
- Range: 0 to 1
- Higher values indicate better matches
- Typically, scores above 0.7 are considered good matches
- Scores above 0.85 are very relevant

**Ordering:**
- Results are ordered by similarity score (highest first)
- If similarity scores are equal, more recent memos appear first

**Empty Results:**
- If no matches found, returns empty array: `[]`
- This is not an error condition

### Notes

- Search uses semantic similarity (embeddings), not exact text matching
- Searches across `content` field primarily
- Also considers tags and metadata for matching
- Language-agnostic (works with English, French, etc.)
- Search is scoped to your organization and project

---

## memoai_memo_record

Record a new memo in your knowledge base.

### Request Schema

```typescript
{
  content: string;      // Required: The memo content
  source?: string;      // Optional: Source identifier
  metadata?: object;    // Optional: Additional key-value metadata
}
```

### Response Schema

```typescript
{
  content: [
    {
      type: "text",
      text: string  // Success message with memo ID
    }
  ]
}
```

### Example Request

```javascript
memoai_memo_record({
  content: "Fixed DuckDB timeout by increasing pool_size to 20. This solved the connection leak issue.",
  source: "manual",
  metadata: {
    file: "src/core/store.py",
    severity: "high"
  }
})
```

### Example Response

```json
{
  "content": [
    {
      "type": "text",
      "text": "Memo recorded successfully: 123e4567-e89b-12d3-a456-426614174000"
    }
  ]
}
```

### Field Details

**content** (required):
- Type: `string`
- Max length: No explicit limit, but keep reasonable (< 5000 characters recommended)
- Should be descriptive and include:
  - What the problem was
  - What the solution is
  - Why it works

**source** (optional):
- Type: `string`
- Common values:
  - `"manual"` - Direct user input
  - `"meeting"` - From team meetings
  - `"code-review"` - From code review feedback
  - `"incident"` - Post-incident learnings
  - `"architecture-review"` - Architecture decisions
- Custom values allowed

**metadata** (optional):
- Type: `object` (key-value pairs)
- Keys: Any string
- Values: Any JSON-serializable type (string, number, boolean, array, object)
- Common metadata fields:
  ```typescript
  {
    file?: string;           // Related file path
    files?: string;          // Comma-separated file paths
    severity?: string;       // "low" | "medium" | "high" | "critical"
    ticket?: string;         // JIRA ticket, GitHub issue, etc.
    author?: string;         // Person who created the learning
    team?: string;           // Team responsible
    project?: string;        // Related project
    date?: string;           // ISO 8601 date
    [key: string]: any;      // Any custom fields
  }
  ```

### Notes

- Episode type is automatically set to `"other"` initially
- The Gardener worker will reclassify it asynchronously
- Classification typically happens within seconds to minutes
- The memo is immediately searchable (even before classification)
- All memos are scoped to your organization and project

---

## Error Responses

### Common Error Formats

All errors follow this format:

```typescript
{
  error: {
    type: string;      // Error type
    message: string;   // Human-readable error message
    details?: object;  // Optional additional details
  }
}
```

### Error Types

#### 1. Missing Required Parameter

**Trigger:** Required parameter not provided

```json
{
  "error": {
    "type": "invalid_request",
    "message": "Missing required parameter: query"
  }
}
```

#### 2. Invalid Parameter Type

**Trigger:** Parameter has wrong type

```json
{
  "error": {
    "type": "invalid_request",
    "message": "Parameter 'limit' must be a number"
  }
}
```

#### 3. Parameter Out of Range

**Trigger:** Parameter value outside valid range

```json
{
  "error": {
    "type": "invalid_request",
    "message": "Parameter 'limit' must be between 1 and 100"
  }
}
```

#### 4. Missing Project Header

**Trigger:** `MEMOAI_PROJECT` environment variable not configured

```json
{
  "error": {
    "type": "configuration_error",
    "message": "MEMOAI_PROJECT header is required. Please configure the MEMOAI_PROJECT environment variable in your MCP client settings."
  }
}
```

#### 5. Authentication Error

**Trigger:** Invalid or expired OAuth token

```json
{
  "error": {
    "type": "authentication_error",
    "message": "Invalid or expired authentication token"
  }
}
```

#### 6. Project Not Found

**Trigger:** Project doesn't exist or user doesn't have access

```json
{
  "error": {
    "type": "not_found",
    "message": "Project 'my-project' not found or access denied"
  }
}
```

#### 7. Internal Server Error

**Trigger:** Unexpected server error

```json
{
  "error": {
    "type": "internal_error",
    "message": "An internal error occurred. Please try again.",
    "details": {
      "request_id": "req_123abc"
    }
  }
}
```

### HTTP Status Codes

| Status | Meaning | When It Occurs |
|--------|---------|----------------|
| 200 | Success | Request completed successfully |
| 400 | Bad Request | Invalid parameters or request format |
| 401 | Unauthorized | Missing or invalid authentication |
| 403 | Forbidden | User doesn't have permission |
| 404 | Not Found | Resource (project, memo) not found |
| 500 | Internal Server Error | Server-side error |
| 503 | Service Unavailable | Service temporarily unavailable |

---

## Internal API Details

The MCP Gateway proxies requests to the Python Core Service. Understanding this helps with debugging.

### Architecture

```
MCP Client (OpenCode/Claude Desktop)
  ↓ OAuth 2.0 (Clerk)
MCP Gateway (TypeScript/Next.js)
  ↓ Internal API (JWT + X-Internal-Key)
Core Service (Python/FastAPI)
  ↓
PostgreSQL + ChromaDB
```

### Request Flow

1. **MCP Client → MCP Gateway**
   - OAuth 2.0 authentication via Clerk
   - `MEMOAI_PROJECT` header required

2. **MCP Gateway → Core Service**
   - Enriched JWT with user context:
     - `user_id` (from Clerk `sub` claim)
     - `org_id` (from Clerk organization memberships)
     - `project_id` (from `MEMOAI_PROJECT` header)
     - `email` (user email)
   - `X-Internal-Key` header for service-to-service auth

3. **Core Service Processing**
   - Validates JWT and internal key
   - Scopes all queries by `org_id` and `project_id`
   - For search: Queries ChromaDB for semantic search
   - For record: Saves to PostgreSQL, queues for embedding

4. **Response Flow**
   - Core Service → MCP Gateway (JSON)
   - MCP Gateway → MCP Client (MCP protocol format)

### Core API Endpoints

**Search Memos:**
```
POST /internal/memos/search
Authorization: Bearer <enriched-jwt>
X-Internal-Key: <shared-secret>

{
  "query": "search query",
  "limit": 10
}
```

**Record Memo:**
```
POST /internal/memos/record
Authorization: Bearer <enriched-jwt>
X-Internal-Key: <shared-secret>

{
  "content": "memo content",
  "source": "manual",
  "metadata": {}
}
```

### Multi-Tenancy

All data is isolated by:

1. **Organization (`org_id`)**
   - Extracted from Clerk organization memberships
   - All queries filtered by `org_id`

2. **Project (`project_id`)**
   - From `MEMOAI_PROJECT` environment variable
   - Allows multiple isolated projects per organization

3. **User (`user_id`)**
   - Tracked for audit purposes
   - Not used for filtering (users within org share data)

### Background Processing

After recording a memo:

1. **Immediate:** Saved to PostgreSQL with `episode_type="other"`
2. **Async (< 1s):** Embedding generated and stored in ChromaDB
3. **Async (< 1min):** Gardener worker classifies and updates `episode_type`
4. **Async (< 5min):** Named entities extracted and linked

### Debugging Tips

**Check authentication:**
```bash
# Verify OAuth token is valid
curl -H "Authorization: Bearer $TOKEN" https://your-mcp-gateway.com/health
```

**Check project configuration:**
```bash
# Verify MEMOAI_PROJECT is set in MCP client config
# Look for error: "MEMOAI_PROJECT header is required"
```

**Check embeddings:**
```bash
# If search returns no results, check if embeddings were generated
# Check Core Service logs for embedding generation errors
```

**Check Gardener classification:**
```bash
# If episode_type is still "other" after several minutes
# Check Core Service logs for Gardener worker errors
```

---

## Response Time Expectations

| Operation | Typical Response Time | Notes |
|-----------|----------------------|-------|
| `memo_search` | 100-500ms | Depends on result count and embedding similarity computation |
| `memo_record` | 50-200ms | Initial save only; embedding/classification happens async |
| Embedding generation | 500ms-2s | Async after recording |
| Episode classification | 1s-30s | Async via Gardener worker |
| Entity extraction | 2s-60s | Async via Gardener worker |

### Performance Tips

1. **Limit search results:** Use smaller `limit` values for faster responses
2. **Cache searches:** If repeatedly searching the same query, cache results client-side
3. **Batch recordings:** If recording multiple memos, send requests in parallel
4. **Monitor timeouts:** Set appropriate timeouts (recommend 10s for search, 5s for record)

---

## Version Information

- **MCP Protocol Version:** `2025-11-05`
- **MCP Gateway Version:** `1.0.0`
- **Tool Schema Version:** `1.0`

---

## Additional Resources

- [Common Workflows](WORKFLOWS.md) - Examples of using these APIs in practice
- [MCP Protocol Spec](https://modelcontextprotocol.io/specification) - Official MCP documentation
