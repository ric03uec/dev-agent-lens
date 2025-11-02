# Phoenix Session Filter Cheatsheet

Quick reference for filtering traces by session metadata in Phoenix UI.

## Session Attributes

| Attribute | Description | Example Values |
|-----------|-------------|----------------|
| `session.id` | Unique session identifier | `code-review-20251101-223845-c835a7` |
| `session.type` | Workflow category | `code-review`, `debugging`, `testing`, `documentation` |
| `session.prompt_version` | Prompt iteration | `v1`, `v2`, `v3` |
| `session.name` | Descriptive label | `oauth-test`, `auth-module-review` |

## Basic Filters

### Filter by Session Type
```
session.type = 'code-review'
```

### Filter by Prompt Version
```
session.prompt_version = 'v2'
```

### Filter by Session Name
```
session.name = 'oauth-test'
```

### Filter by Specific Session
```
session.id = 'testing-20251101-223542-a1edb5'
```

## Compound Filters

### Code Reviews with Prompt v2
```
session.type = 'code-review' AND session.prompt_version = 'v2'
```

### Specific Project Across Versions
```
session.name = 'auth-refactor' AND session.prompt_version IN ('v1', 'v2', 'v3')
```

### All Sessions from Today
```
session.type = 'code-review' AND timestamp > '2025-11-01'
```

### Exclude Testing Sessions
```
session.type != 'testing'
```

## Grouping & Aggregation

### Group by Session Type
**Use Case:** Compare costs across workflow types

1. Group by: `session.type`
2. View: Total cost, average tokens, trace count

### Group by Prompt Version
**Use Case:** Measure prompt iteration improvements

1. Filter: `session.type = 'code-review' AND session.name = 'auth-module'`
2. Group by: `session.prompt_version`
3. Compare: Token usage, latency, cost

### Group by Session Name
**Use Case:** Track specific projects over time

1. Group by: `session.name`
2. View: Total traces, cumulative cost

## Common Workflows

### 1. Compare Two Prompt Versions

**Step 1:** Run both versions
```bash
./claude-session --type code-review --version v1 --name "prompt-test"
claude code "Review auth.py"

./claude-session --type code-review --version v2 --name "prompt-test"
claude code "Review auth.py for security, performance, and best practices"
```

**Step 2:** Filter in Phoenix
```
session.name = 'prompt-test'
```

**Step 3:** Group by
```
session.prompt_version
```

**Step 4:** Compare
- Token usage: v1 vs v2
- Latency: Which is faster?
- Cost: Which is cheaper?

### 2. Find Most Expensive Workflow

**Step 1:** Group by
```
session.type
```

**Step 2:** Sort by
- Total cost (descending)

**Step 3:** Identify
- Which workflow type costs most?
- Optimize those prompts first

### 3. Track Project Progress

**Step 1:** Filter by project
```
session.name = 'async-refactor'
```

**Step 2:** View timeline
- All traces for this project
- Cost accumulation over time
- Token usage trends

### 4. Debug Specific Session

**Step 1:** Get session ID
```bash
docker exec dev-agent-lens-litellm-proxy-phoenix-1 env | grep PHOENIX_SESSION_ID
```

**Step 2:** Filter in Phoenix
```
session.id = 'code-review-20251101-223845-c835a7'
```

**Step 3:** Analyze
- All traces from this session
- Error rates
- Performance issues

## Filter Operators

| Operator | Example | Description |
|----------|---------|-------------|
| `=` | `session.type = 'debugging'` | Equals |
| `!=` | `session.type != 'testing'` | Not equals |
| `IN` | `session.prompt_version IN ('v2', 'v3')` | In list |
| `>` | `timestamp > '2025-11-01'` | Greater than |
| `<` | `latency < 1000` | Less than |
| `AND` | `session.type = 'code-review' AND session.prompt_version = 'v2'` | Both conditions |
| `OR` | `session.type = 'debugging' OR session.type = 'testing'` | Either condition |

## Useful Metrics

When viewing filtered sessions, check:

| Metric | What It Shows | Optimize For |
|--------|---------------|--------------|
| **Total Tokens** | Token usage across sessions | Lower = more efficient prompts |
| **Avg Latency** | Response time | Lower = faster responses |
| **Total Cost** | API costs | Lower = cheaper operations |
| **Trace Count** | Number of requests | Track usage patterns |
| **Error Rate** | Failed requests | Lower = more reliable |

## Quick Commands

### Check Current Session
```bash
docker exec dev-agent-lens-litellm-proxy-phoenix-1 env | grep PHOENIX_SESSION
```

### List All Session Types (from logs)
```bash
docker logs dev-agent-lens-litellm-proxy-phoenix-1 2>&1 | grep "session.type" | sort -u
```

### Reset Session
```bash
./claude-session --reset
```

## Tips

1. **Use Descriptive Names**: `session.name = 'auth-security-audit'` > `session.name = 'test1'`

2. **Version Your Prompts**: Use semantic versions (v1, v2, v3) to track iterations

3. **Start New Sessions**: Use `./claude-session --reset` between different tasks

4. **Track Experiments**: Same `session.name`, different `session.prompt_version`

5. **Group by Type**: See overall workflow costs

6. **Group by Version**: Measure prompt improvements

7. **Group by Name**: Track project-specific costs

## Example Analysis

### Find Most Efficient Prompt Version

```
Filter: session.name = 'code-review-test'
Group By: session.prompt_version
Sort By: Average tokens (ascending)

Results:
v3: 1,200 tokens avg ($0.012)  ← Most efficient!
v2: 1,800 tokens avg ($0.018)
v1: 2,500 tokens avg ($0.025)
```

**Action:** Use v3 for all future code reviews!

### Identify Cost Hotspots

```
Group By: session.type
Sort By: Total cost (descending)

Results:
code-review:    $5.40 (50 traces)  ← Most expensive
debugging:      $2.30 (35 traces)
documentation:  $1.20 (20 traces)
testing:        $0.80 (15 traces)
```

**Action:** Optimize code-review prompts to reduce costs!

## Phoenix UI Navigation

1. **Open Phoenix**: http://localhost:6006
2. **Click "Traces"** in left sidebar
3. **Use filter bar** at top of traces view
4. **Type filter query** (e.g., `session.type = 'code-review'`)
5. **Press Enter** to apply
6. **Use "Group By"** dropdown for aggregation
7. **Click individual traces** to see details
8. **View spans** to see request/response flow

## Troubleshooting

### Filter Not Working?

**Check attribute name spelling:**
- ✅ `session.type` (lowercase)
- ❌ `session.Type` (wrong case)
- ❌ `sessionType` (wrong format)

**Check quotes:**
- ✅ `session.type = 'code-review'` (single quotes)
- ❌ `session.type = code-review` (missing quotes)

### No Results?

**Verify session is set:**
```bash
docker exec dev-agent-lens-litellm-proxy-phoenix-1 env | grep PHOENIX_SESSION
```

**Check OTEL attributes:**
```bash
docker exec dev-agent-lens-litellm-proxy-phoenix-1 env | grep OTEL_RESOURCE_ATTRIBUTES
```

Should include: `session.id=...,session.type=...,session.prompt_version=...`

### Session Not Updating?

**Restart container with new session:**
```bash
./claude-session --type debugging --version v1
```

This automatically restarts the container with new session metadata.

---

**Quick Reference:**
- Filter: `session.type = 'code-review'`
- Group: Select `session.prompt_version` from dropdown
- Compare: View token usage, cost, latency across versions
- Iterate: Optimize prompts based on metrics
