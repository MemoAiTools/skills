# Common Workflows

Step-by-step workflows for using MemoAI effectively in common scenarios.

## Table of Contents

1. [Workflow 1: Debugging with Memory Search](#workflow-1-debugging-with-memory-search)
2. [Workflow 2: Recording Learnings After Bug Fixes](#workflow-2-recording-learnings-after-bug-fixes)
3. [Workflow 3: Proactive Search Before Implementation](#workflow-3-proactive-search-before-implementation)
4. [Workflow 4: Team Knowledge Sharing](#workflow-4-team-knowledge-sharing)
5. [Workflow 5: Post-Incident Learning](#workflow-5-post-incident-learning)
6. [Workflow 6: Architecture Decision Recording](#workflow-6-architecture-decision-recording)

---

## Workflow 1: Debugging with Memory Search

**Scenario:** User encounters an error and asks if you've seen it before.

### Steps

1. **User reports issue**
   ```
   User: "I'm getting a DuckDB timeout error when running queries. Have we seen this before?"
   ```

2. **Search memory proactively**
   ```javascript
   memoai_memo_search({
     query: "DuckDB timeout error",
     limit: 5
   })
   ```

3. **Analyze results**
   - Review returned memos for relevant solutions
   - Look for similar symptoms or error messages
   - Identify applicable fixes

4. **Provide informed answer**
   ```
   Agent: "Yes, we've encountered this before! Found 2 relevant learnings:
   
   1. Fixed timeout by increasing pool_size to 20 (from default 10)
   2. Always use context manager to release connections promptly
   
   The issue is typically caused by connection leaks when queries fail. 
   Try increasing pool_size in your DuckDB connection config and ensure 
   you're using 'with store.connection() as conn:' pattern."
   ```

5. **If solution works, offer to enhance the record**
   ```
   Agent: "Great! Should I update our memory with this specific case?"
   ```

### Tips

- Search before answering, even if you think you know the solution
- Use the exact error message in your search query
- Include technology names (DuckDB, Redis, etc.) in queries
- If no results, still record the new solution for future reference

---

## Workflow 2: Recording Learnings After Bug Fixes

**Scenario:** User reports they fixed a bug and you want to record the learning.

### Steps

1. **User describes the fix**
   ```
   User: "I fixed the timeout by increasing pool_size to 20. This solved the connection leak issue."
   ```

2. **Detect the learning pattern**
   - Pattern detected: "I fixed" + technical details
   - Type: Bug fix
   - Content is sufficiently detailed

3. **Confirm with user**
   ```
   Agent: "Should I record this bug fix in your memory?
   
   Suggested content:
   'Fixed DuckDB timeout by increasing pool_size to 20. This solved the connection leak issue.'
   
   [Yes] [Edit first] [No]"
   ```

4. **If Yes, record the memo**
   ```javascript
   memoai_memo_record({
     content: "Fixed DuckDB timeout by increasing pool_size to 20. This solved the connection leak issue.",
     source: "manual"
   })
   ```

5. **Confirm success**
   ```
   Agent: "Recorded successfully! This will be available for future searches."
   ```

### Tips

- Always ask for confirmation before recording
- Allow users to edit the content before recording
- Include the "what", "how", and "why" in the content
- Don't record trivial fixes (e.g., "fixed typo")

---

## Workflow 3: Proactive Search Before Implementation

**Scenario:** User is about to start implementing a feature.

### Steps

1. **User announces task**
   ```
   User: "I need to optimize the embedding batch processing"
   ```

2. **Search for relevant past work**
   ```javascript
   memoai_memo_search({
     query: "optimize embedding batch processing performance",
     limit: 5
   })
   ```

3. **Share relevant findings**
   ```
   Agent: "Let me check if we have relevant past experiences...
   
   Found 2 relevant learnings:
   
   1. 'Optimized embedding batching with size=100, achieved 3x speedup. 
      Using concurrent processing with asyncio.gather().'
   
   2. 'Avoid recomputing embeddings for existing content. Cache embeddings 
      in Redis with TTL of 24 hours.'
   
   Based on these learnings, here's a suggested approach..."
   ```

4. **Implement with context**
   - Use the learnings to inform your implementation
   - Avoid repeating past mistakes
   - Build on successful patterns

5. **After completion, record new insights**
   ```javascript
   memoai_memo_record({
     content: "Improved embedding batch processing from 5s to 1s by combining two approaches: batch size of 100 (from previous learning) and parallel processing with 4 workers. The key insight was that DuckDB writes are bottleneck, not embedding generation.",
     source: "manual"
   })
   ```

### Tips

- Search before implementation, not after
- Share findings with user to get their input
- Combine multiple past learnings when applicable
- Record new insights that weren't in previous learnings

---

## Workflow 4: Team Knowledge Sharing

**Scenario:** Recording knowledge from team meetings or code reviews.

### Steps

1. **Capture the decision or learning**
   ```
   User: "In today's architecture review, we decided to use FastAPI instead of Flask for the new API service because of better async support."
   ```

2. **Confirm recording**
   ```
   Agent: "Should I record this architecture decision?
   
   Suggested content:
   'Use FastAPI instead of Flask for new API services. Decision made in architecture review due to better async support and performance with concurrent requests.'
   
   [Yes] [Edit first] [No]"
   ```

3. **Record with source and metadata**
   ```javascript
   memoai_memo_record({
     content: "Use FastAPI instead of Flask for new API services. Decision made in architecture review due to better async support and performance with concurrent requests.",
     source: "architecture-review",
     metadata: {
       meeting_date: "2024-01-15",
       participants: "engineering-team",
       context: "new-api-service"
     }
   })
   ```

4. **Confirm and suggest related searches**
   ```
   Agent: "Recorded! In the future, team members can search for 'API framework decision' or 'FastAPI vs Flask' to find this context."
   ```

### Tips

- Use descriptive source values (e.g., "architecture-review", "code-review", "team-meeting")
- Include context in metadata (meeting date, participants, related project)
- Record decisions with their rationale, not just the outcome
- Make content searchable by including alternative phrasings

---

## Workflow 5: Post-Incident Learning

**Scenario:** Recording learnings after a production incident.

### Steps

1. **Incident resolved**
   ```
   User: "We had a production outage caused by Redis connection exhaustion. Fixed by increasing max connections to 100 and adding connection timeout of 5s."
   ```

2. **Record comprehensive learning**
   ```javascript
   memoai_memo_record({
     content: "Production outage caused by Redis connection exhaustion. Redis was configured with default max_clients=10, insufficient for our traffic. Fixed by: (1) Increasing max_clients to 100 in redis.conf, (2) Adding connection timeout of 5s to prevent hanging connections, (3) Implementing connection pooling with max_size=50 in application. Monitoring showed 30-40 active connections during peak traffic.",
     source: "incident",
     metadata: {
       severity: "high",
       incident_id: "INC-2024-015",
       date: "2024-01-20",
       impact: "15min-downtime",
       services: "api,worker"
     }
   })
   ```

3. **Confirm and suggest preventive searches**
   ```
   Agent: "Recorded with high severity. In the future, searches for 'Redis connection issues' or 'production outage' will surface this learning."
   ```

### Tips

- Include root cause, not just symptoms
- Document the fix with specific parameters
- Add severity and impact in metadata
- Link to incident IDs or post-mortems
- Record preventive measures, not just reactive fixes

---

## Workflow 6: Architecture Decision Recording

**Scenario:** Recording a significant architectural choice.

### Steps

1. **Decision made**
   ```
   User: "After evaluating options, we'll use ChromaDB for vector storage instead of Pinecone. Main reasons: (1) Self-hosted for data privacy, (2) Lower cost at our scale, (3) Good performance for our use case."
   ```

2. **Record with full context**
   ```javascript
   memoai_memo_record({
     content: "Architecture decision: Use ChromaDB for vector storage instead of Pinecone. Rationale: (1) Self-hosted deployment meets our data privacy requirements - customer data never leaves our infrastructure, (2) Lower cost at current scale (100K vectors) - estimated $0 vs $70/month for Pinecone, (3) Performance is sufficient for our use case - 50ms average query latency meets our <100ms SLA. Trade-offs accepted: Manual scaling and maintenance overhead, less feature-rich than Pinecone (no advanced filtering). Alternative considered: Pinecone (rejected due to cost and privacy), Weaviate (rejected due to complexity).",
     source: "architecture-review",
     metadata: {
       decision_date: "2024-01-15",
       alternatives: "pinecone,weaviate",
       category: "infrastructure",
       impact: "high"
     }
   })
   ```

3. **Confirm and suggest tags**
   ```
   Agent: "Recorded architecture decision. Tagged with alternatives considered so future searches for 'Pinecone' or 'Weaviate' will also surface this decision."
   ```

### Tips

- Include the "why", not just the "what"
- Document alternatives considered and why they were rejected
- Be explicit about trade-offs accepted
- Add business context (cost, privacy, compliance)
- Include performance metrics or SLAs when relevant
- Use metadata to link to related decisions

---

## General Workflow Tips

### Search Query Strategies

**1. Use specific technology names**
```
Good: "DuckDB timeout error"
Poor: "database timeout"
```

**2. Include action verbs**
```
Good: "optimize embedding batch processing"
Poor: "embeddings"
```

**3. Use natural language**
```
Good: "how to handle connection leaks"
Good: "comment gérer les fuites de connexion"
```

**4. Include error messages**
```
Good: "OSError: [Errno 24] Too many open files"
```

### Recording Content Best Practices

**1. Use action-oriented openings**
```
Good: "Fixed timeout by..."
Good: "Implemented authentication using..."
Good: "Refactored database layer to..."
```

**2. Include the problem and solution**
```
Good: "Fixed memory leak in cache. The cache wasn't expiring old entries. Added TTL of 1 hour."
Poor: "Fixed cache"
```

**3. Add specific values**
```
Good: "Increased pool_size from 10 to 20"
Poor: "Increased pool_size"
```

**4. Explain the why**
```
Good: "Use FastAPI for async endpoints because it handles concurrent requests better than Flask"
Poor: "Use FastAPI"
```

### Metadata Strategies

**1. Related files**
```javascript
metadata: {
  files: "src/core/store.py,src/api/routes.py"
}
```

**2. Severity or priority**
```javascript
metadata: {
  severity: "high",
  priority: "urgent"
}
```

**3. Team or person**
```javascript
metadata: {
  author: "jane-doe",
  team: "backend"
}
```

**4. Project or context**
```javascript
metadata: {
  project: "api-v2",
  sprint: "2024-Q1-S3"
}
```

**5. External references**
```javascript
metadata: {
  ticket: "JIRA-1234",
  pr: "github.com/org/repo/pull/567",
  doc: "https://..."
}
```

---

## Troubleshooting

### Search Returns No Results

**Problem:** Search query returns empty results even though you know the information was recorded.

**Solutions:**

1. **Try different phrasings**
   - "DuckDB timeout" vs "database connection timeout"
   - "optimize performance" vs "speed up queries"

2. **Use broader queries**
   - "DuckDB" (broad) vs "DuckDB pool_size configuration" (narrow)

3. **Search for technology names**
   - "Redis" will find all Redis-related learnings

4. **Check if the memo was actually recorded**
   - Verify that recording completed successfully
   - Classification happens async, but search works immediately

### Too Many Irrelevant Results

**Problem:** Search returns too many results, making it hard to find relevant information.

**Solutions:**

1. **Be more specific**
   - Add technology names, error messages, or specific terms
   - Use longer queries with more context

2. **Reduce limit**
   ```javascript
   memoai_memo_search({ query: "...", limit: 3 })
   ```

3. **Use compound queries**
   - "DuckDB timeout production" (multiple specific terms)

### User Declines Recording

**Problem:** User says "No" when you ask to record a learning.

**Solutions:**

1. **Respect the decision** - Don't push back
2. **Consider offering again later** if they provide more details
3. **Learn patterns** - Some users prefer to record manually

---

## Advanced Patterns

### Multi-Step Problem Solving

When solving complex problems, use this pattern:

1. **Initial search** - Check for similar problems
2. **Work through solution** - Implement and test
3. **Record outcome** - Include what worked and what didn't
4. **Link related learnings** - Use metadata to connect related memos

### Knowledge Evolution

When updating existing knowledge:

1. **Record new learning** - Don't edit the old one
2. **Reference previous learning** - Mention in content or metadata
3. **Explain evolution** - Why did understanding change?

Example:
```javascript
memoai_memo_record({
  content: "Updated approach for DuckDB connection pooling. Previous learning suggested pool_size=20, but found that pool_size=50 is optimal for our production load (handles 1000 req/s vs 600 req/s with pool_size=20). Key insight: pool_size should scale with concurrent request volume, not just connection timeout issues.",
  metadata: {
    supersedes: "previous learning about pool_size=20",
    context: "production optimization"
  }
})
```
