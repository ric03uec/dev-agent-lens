# Phoenix Session Tracking for Claude Code

## Overview

Session tracking allows you to organize and analyze Claude Code usage by:
- **Workflow Type** (code-review, debugging, documentation, testing, etc.)
- **Prompt Version** (v1, v2, v3, etc.)
- **Session Name** (optional descriptive label)

This enables you to:
- ✅ **Track cost and latency** per workflow type
- ✅ **Compare prompt versions** side-by-side
- ✅ **Iterate on prompts** and measure improvements
- ✅ **Filter traces** by session metadata in Phoenix UI

## Quick Start

### 1. Start a Code Review Session

```bash
./claude-session --type code-review --version v2 --name "async-refactor"
```

This will:
1. Generate a unique session ID
2. Restart the LiteLLM proxy with session metadata
3. Start an interactive shell with the session active

All Claude Code requests in this shell will be tagged with the session metadata.

### 2. Run Claude Code Commands

```bash
# In the interactive session shell
claude code "Review this authentication module for security issues"
claude code "Suggest optimizations for the database query in users.py"
```

### 3. View Traces in Phoenix

Open Phoenix UI: http://localhost:6006

Filter by:
- `session.type = "code-review"`
- `session.prompt_version = "v2"`
- `session.name = "async-refactor"`

### 4. Reset Session When Done

```bash
./claude-session --reset
```

This clears the session metadata and returns to default tracking.

## Usage Examples

### Example 1: Compare Prompt Versions

Test two different prompts for the same task:

```bash
# Version 1 - Basic prompt
./claude-session --type code-review --version v1 --name "auth-module"
claude code "Review the authentication module"
exit

# Version 2 - Detailed prompt
./claude-session --type code-review --version v2 --name "auth-module"
claude code "Review the authentication module for: 1) Security vulnerabilities 2) Performance issues 3) Code quality 4) Best practices"
exit
```

Compare in Phoenix:
- Filter by `session.name = "auth-module"`
- Group by `session.prompt_version`
- Compare token usage, cost, and latency

### Example 2: Track Different Workflows

Separate sessions for different types of work:

```bash
# Debugging session
./claude-session --type debugging --version v1 --shell

# Documentation session
./claude-session --type documentation --version v3 --shell

# Testing session
./claude-session --type testing --version v2 --shell
```

In Phoenix, filter by `session.type` to see metrics per workflow type.

### Example 3: Quick One-Off Command

Run a single Claude Code command with session tracking:

```bash
./claude-session --type code-review --version v2 "Review the auth module for security issues"
```

The session is active only for this one command.

## Command Reference

### Options

```bash
./claude-session [OPTIONS] [CLAUDE_COMMAND]

Options:
  -t, --type TYPE        Session type (e.g., code-review, debugging, documentation)
                         Default: general

  -v, --version VERSION  Prompt version (e.g., v1, v2, v3)
                         Default: v1

  -n, --name NAME        Optional descriptive name for this session
                         (e.g., "async-refactor", "api-docs-v2")

  -s, --shell            Start interactive shell with session active

  -r, --reset            Reset to default session (clears session metadata)

  -h, --help             Show help message
```

### Session Metadata

Each session is tagged with:

| Attribute | Description | Example |
|-----------|-------------|---------|
| `session.id` | Unique session identifier | `code-review-20251101-143022-a3f2b1` |
| `session.type` | Category of work | `code-review`, `debugging`, `documentation` |
| `session.prompt_version` | Your prompt iteration version | `v1`, `v2`, `v3` |
| `session.name` | Optional descriptive label | `async-refactor`, `api-docs-v2` |

## Recommended Workflow Types

Organize your work into these common types:

- **code-review** - Code review and analysis
- **debugging** - Debugging and troubleshooting
- **documentation** - Writing documentation
- **testing** - Test creation and analysis
- **refactoring** - Code refactoring
- **architecture** - Architecture design and planning
- **learning** - Learning and exploration
- **general** - General purpose (default)

Or create your own types based on your workflow!

## Prompt Version Iteration

Use semantic versioning for prompts:

```bash
# v1 - Initial prompt
./claude-session --type code-review --version v1
claude code "Review this code"

# v2 - Add specific criteria
./claude-session --type code-review --version v2
claude code "Review this code for security, performance, and maintainability"

# v3 - Add structure
./claude-session --type code-review --version v3
claude code "Review this code and provide:
1. Security analysis
2. Performance bottlenecks
3. Maintainability concerns
4. Specific recommendations"
```

Track metrics across versions to measure improvement:
- Token usage (lower = more efficient)
- Response quality (subjective)
- Latency (faster = better UX)
- Cost (lower = cheaper)

## Phoenix UI Filtering

### Filter by Session Type

```
session.type = "code-review"
```

### Filter by Prompt Version

```
session.prompt_version = "v2"
```

### Filter by Session Name

```
session.name = "async-refactor"
```

### Combined Filters

```
session.type = "code-review" AND session.prompt_version = "v2"
```

### Group By

Group traces to see aggregated metrics:
- Group by `session.type` - See metrics per workflow
- Group by `session.prompt_version` - Compare prompt versions
- Group by `session.name` - Track specific projects

## Best Practices

### 1. Use Descriptive Session Names

```bash
# Good
./claude-session --type code-review --version v2 --name "auth-security-audit"

# Less useful
./claude-session --type code-review --version v2 --name "test1"
```

### 2. Increment Prompt Versions

When you change your prompt significantly:

```bash
# First attempt
./claude-session --type debugging --version v1 "Find the bug"

# More specific
./claude-session --type debugging --version v2 "Find the bug causing the timeout in user login"

# Even more structured
./claude-session --type debugging --version v3 "Analyze the user login flow and identify:
1. Timeout causes
2. Database query performance
3. Network latency issues"
```

### 3. Reset After Each Session

```bash
# Work on task
./claude-session --type code-review --version v2 --shell
# ... do work ...
exit

# Reset for next task
./claude-session --reset

# Start new session
./claude-session --type debugging --version v1 --shell
```

### 4. Track A/B Tests

Compare different approaches:

```bash
# Approach A - Detailed instructions
./claude-session --type code-review --version v2 --name "approach-a-detailed"
claude code "Detailed prompt here..."

# Approach B - Minimal instructions
./claude-session --type code-review --version v2 --name "approach-b-minimal"
claude code "Minimal prompt here..."
```

Compare metrics in Phoenix to see which approach is more efficient.

## Advanced Usage

### Manual Session Management

You can also set session variables manually:

```bash
export PHOENIX_SESSION_ID="my-custom-id-$(date +%s)"
export PHOENIX_SESSION_TYPE="custom-workflow"
export PHOENIX_PROMPT_VERSION="v1"
export PHOENIX_SESSION_NAME="experimental"

# Restart container
cd /path/to/dev-agent-lens
docker compose --profile phoenix up -d --force-recreate litellm-proxy-phoenix

# Use Claude Code
claude code "your prompt"
```

### Session Templates

Create shell scripts for common sessions:

```bash
# ~/bin/claude-code-review
#!/bin/bash
cd /path/to/dev-agent-lens
./claude-session --type code-review --version v2 --shell

# ~/bin/claude-debugging
#!/bin/bash
cd /path/to/dev-agent-lens
./claude-session --type debugging --version v1 --shell
```

### Persistent Sessions

By default, sessions reset when you exit. To keep a session active:

```bash
./claude-session --type code-review --version v2 --name "long-running"

# Don't reset - just exit the shell
# Session remains active for subsequent Claude Code commands
```

When done:
```bash
./claude-session --reset
```

## Troubleshooting

### Session Not Appearing in Phoenix

**Check:**
1. Container is running: `docker ps | grep litellm`
2. Session vars are set: `docker exec dev-agent-lens-litellm-proxy-phoenix-1 env | grep PHOENIX`
3. OTEL attributes include session: Look for `session.id`, `session.type` in container logs

**Verify:**
```bash
# Check current session
docker exec dev-agent-lens-litellm-proxy-phoenix-1 env | grep PHOENIX_SESSION

# Should show:
# PHOENIX_SESSION_ID=code-review-20251101-...
# PHOENIX_SESSION_TYPE=code-review
# PHOENIX_PROMPT_VERSION=v2
```

### Container Not Restarting

**Error:** `docker compose` command fails

**Fix:**
```bash
cd /path/to/dev-agent-lens
docker compose --profile phoenix down
docker compose --profile phoenix up -d
```

### Session ID Not Unique

**Issue:** Multiple sessions with same ID

**Cause:** Container not restarted between sessions

**Fix:** Always use `claude-session` script which handles restarts automatically

## Integration with CI/CD

Track CI/CD pipeline metrics:

```bash
# In your CI script
export PHOENIX_SESSION_TYPE="ci-pipeline"
export PHOENIX_SESSION_ID="build-${CI_BUILD_ID}"
export PHOENIX_PROMPT_VERSION="v1"

# Restart proxy (in CI environment)
docker compose --profile phoenix up -d --force-recreate litellm-proxy-phoenix

# Run Claude Code tasks
claude code "Review this PR for security issues"
```

## Cost Analysis

Track costs per workflow type in Phoenix:

1. Go to Phoenix UI: http://localhost:6006
2. Navigate to "Traces" or "Analytics"
3. Group by `session.type`
4. View total cost per session type
5. Identify expensive workflows
6. Iterate on prompts to reduce costs

Example insights:
- "code-review v1 uses 2000 tokens on average, v2 uses 1500 tokens (25% reduction)"
- "debugging sessions cost $0.10 on average, optimization opportunity identified"
- "documentation prompts are most efficient at 800 tokens average"

## Technical Implementation

### How Session Tracking Works

Session metadata flows through the following pipeline:

1. **Environment Variables** → Container receives `PHOENIX_SESSION_*` environment variables
2. **Logging Injection** → LiteLLM's logging function reads env vars and injects into `metadata['metadata']`
3. **Metadata Merging** → Session data is merged into `requester_metadata` structure
4. **Phoenix Integration** → Arize integration reads `requester_metadata` and sets OpenInference span attributes
5. **OTEL Export** → Span attributes exported to Phoenix via OpenTelemetry

### Modified Files (LiteLLM)

The following files were modified in the LiteLLM codebase to enable session tracking:

#### 1. `litellm/litellm_core_utils/litellm_logging.py`

**Lines 4586-4606**: Direct environment variable injection
```python
# Inject session metadata from environment variables for Phoenix filtering
import os
session_id = os.getenv("PHOENIX_SESSION_ID")
session_type = os.getenv("PHOENIX_SESSION_TYPE")
prompt_version = os.getenv("PHOENIX_PROMPT_VERSION")
session_name = os.getenv("PHOENIX_SESSION_NAME")
if session_id or session_type or prompt_version or session_name:
    if "metadata" not in metadata:
        metadata["metadata"] = {}
    if session_id:
        metadata["metadata"]["session_id"] = session_id
    # ... other fields
```

**Lines 4087-4097**: Metadata merging logic
```python
if _potential_requester_metadata is not None and isinstance(_potential_requester_metadata, dict):
    if clean_metadata["requester_metadata"] is None:
        clean_metadata["requester_metadata"] = _potential_requester_metadata
    else:
        # Merge instead of replace to preserve existing metadata like user_id
        clean_metadata["requester_metadata"].update(_potential_requester_metadata)
```

#### 2. `litellm/integrations/arize/_utils.py`

**Lines 76-97**: Check both direct metadata and requester_metadata
```python
# Session metadata can be in two places:
# 1. metadata["session_id"] (direct)
# 2. metadata["requester_metadata"]["session_id"] (from proxy endpoints)
requester_metadata = metadata.get("requester_metadata", {})

session_id = metadata.get("session_id") or requester_metadata.get("session_id")
if session_id:
    safe_set_attribute(span, SpanAttributes.SESSION_ID, session_id)
```

#### 3. `litellm/proxy/session_metadata_middleware.py`

Middleware that injects session metadata from environment variables into request data (used as fallback).

### OpenInference Specification Compliance

The implementation sets the following OpenInference span attributes:

- `session.id` (required) - Maps to `SpanAttributes.SESSION_ID`
- `session.type` (custom)
- `session.prompt_version` (custom)
- `session.name` (custom)

These attributes are indexed by Phoenix and available for filtering and grouping.

### Debugging Session Tracking

To verify session tracking is working:

```bash
# Check container environment variables
docker exec dev-agent-lens-litellm-proxy-phoenix-1 env | grep PHOENIX_SESSION

# Check logs for injection messages
docker logs dev-agent-lens-litellm-proxy-phoenix-1 2>&1 | grep "LOGGING.*Injected session"

# Check logs for Phoenix span attribute setting
docker logs dev-agent-lens-litellm-proxy-phoenix-1 2>&1 | grep "PHOENIX DEBUG.*session"
```

Expected log output:
```
[LOGGING] Injected session_id from env: test-session-1762066447
[LOGGING] Injected session_type from env: debug-test
[PHOENIX DEBUG] Setting session.id = test-session-1762066447
[PHOENIX DEBUG] session.id attribute set successfully
```

## References

- [Phoenix Session Documentation](https://arize.com/docs/phoenix/tracing/how-to-tracing/setup-tracing/setup-sessions)
- [OpenInference Specification](https://github.com/Arize-ai/openinference)
- [LiteLLM Documentation](https://docs.litellm.ai/)
- [Claude Code Documentation](https://docs.anthropic.com/claude/docs/claude-code)

## Support

For issues or questions:
1. Check container logs: `docker logs dev-agent-lens-litellm-proxy-phoenix-1`
2. Verify Phoenix is running: `curl http://localhost:6006`
3. Check OTLP endpoint: `nc -zv localhost 4317`
4. Review this documentation
5. Check Phoenix UI for trace data

---

**Happy tracking! 🚀**

Use session tracking to iterate on your prompts, optimize costs, and improve your Claude Code workflows.
