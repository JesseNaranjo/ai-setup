# Usage Tracking

This document defines the schema and protocol for tracking agent invocations during code reviews.

## Purpose

Track which agents were actually invoked (confirmed via Task tool returns) and their timing to:
- Monitor agents that take too long (potential issues)
- Detect agents that finish too early (potential failures)
- Provide visibility into review execution for users

## Tracking Schema

Maintain this structure internally during workflow execution:

```yaml
usage_tracking:
  review_type: "deep-review"  # deep-review, deep-review-staged, quick-review, quick-review-staged
  review_started_at: "2024-01-15T10:30:00Z"  # ISO 8601 timestamp
  review_ended_at: "2024-01-15T10:33:05Z"
  total_duration_seconds: 185

  phases:
    - name: "Phase 1: Thorough Review"
      phase_started_at: "2024-01-15T10:30:00Z"
      phase_ended_at: "2024-01-15T10:31:44Z"
      phase_duration_seconds: 104
      agents:
        - name: "compliance-agent"
          model: "sonnet"
          mode: "thorough"
          agent_started_at: "2024-01-15T10:30:00Z"
          agent_ended_at: "2024-01-15T10:30:27Z"
          agent_duration_seconds: 27
          status: "completed"  # completed, failed, skipped
          task_id: "abc123"    # Confirms actual invocation
          findings_count: 3    # Number of issues found by this agent
        - name: "bug-detection-agent"
          model: "opus"
          mode: "thorough"
          agent_started_at: "2024-01-15T10:30:00Z"
          agent_ended_at: "2024-01-15T10:31:44Z"
          agent_duration_seconds: 104
          status: "completed"
          task_id: "def456"
          findings_count: 5
        # ... remaining agents

    - name: "Phase 2: Gaps Review"
      phase_started_at: "2024-01-15T10:31:44Z"
      phase_ended_at: "2024-01-15T10:32:28Z"
      phase_duration_seconds: 44
      agents:
        # ... gaps agents

    - name: "Synthesis"
      phase_started_at: "2024-01-15T10:32:28Z"
      phase_ended_at: "2024-01-15T10:32:57Z"
      phase_duration_seconds: 29
      agents:
        # ... synthesis agents
```

## Recording Protocol

### Initialization

Before launching the first agent:

1. Record `review_started_at` timestamp for the review
2. Initialize the phases array based on review type:
   - **Deep review**: Phase 1 (8 agents), Phase 2 (4 agents), Synthesis (4 agents)
   - **Quick review**: Review (4 agents), Synthesis (3 agents)

### Recording Each Agent Invocation

**Before launching an agent:**
1. Record the agent's `agent_started_at` timestamp (current time)

**After Task tool returns:**
1. Record `agent_ended_at` timestamp (current time)
2. Calculate `agent_duration_seconds` = agent_ended_at - agent_started_at
3. Record the `task_id` from the Task tool return (confirms actual invocation)
4. Set `status` to "completed"
5. Count issues in agent output and record `findings_count`

### Confirming Actual Invocation

The Task tool returns a `task_id` when an agent completes. This ID serves as proof of actual invocation:
- If `task_id` is present: Agent was actually invoked and completed
- If `task_id` is absent but duration exists: Agent may have failed

### Phase Timing

For parallel agent execution within a phase:
- **phase_started_at**: Timestamp when first agent launched
- **phase_ended_at**: Timestamp when last agent completed
- **phase_duration_seconds**: phase_ended_at - phase_started_at
- **Note**: Sum of individual agent durations may exceed phase duration (expected with parallelism)

### Status Values

| Status | Description |
|--------|-------------|
| `completed` | Agent finished successfully with task_id |
| `failed` | Agent errored or returned no results |
| `skipped` | Agent was in skip_agents list |

### Edge Cases

**Skipped Agents (skip_agents setting):**
```yaml
- name: "architecture-agent"
  model: "sonnet"
  mode: "thorough"
  status: "skipped"
  agent_duration_seconds: 0
  findings_count: 0
  # No timestamps or task_id for skipped agents
```

**Failed Agents:**
```yaml
- name: "security-agent"
  model: "opus"
  mode: "thorough"
  agent_started_at: "2024-01-15T10:30:00Z"
  agent_ended_at: "2024-01-15T10:30:15Z"
  agent_duration_seconds: 15
  status: "failed"
  findings_count: 0
  # task_id may be present if failure occurred after invocation
```

**Interrupted Review:**
- Generate partial summary with available data
- Note which phases/agents were incomplete

## Timing Anomaly Detection

Flag potential issues based on timing by agent role:

| Agent Role | Models Used | Expected Range | Too Fast | Too Slow |
|------------|-------------|---------------|----------|----------|
| Review agents (Opus) | opus | 30-120s | < 15s `[!]` | > 180s `[*]` |
| Review agents (Sonnet) | sonnet | 15-60s | < 10s `[!]` | > 120s `[*]` |
| Synthesis agents | sonnet | 10-45s | < 5s `[!]` | > 90s `[*]` |

**Note:** Synthesis agents use Sonnet model but have different expected timing because they analyze aggregated findings rather than raw code. Use the "Agent Role" column to determine which thresholds apply.

**Indicators:**
- `[!]` = Agent may have errored or found nothing (too fast)
- `[*]` = Agent may be stuck or processing large files (too slow)

## Review Type Configurations

### Deep Review (16 invocations)

```yaml
phases:
  - name: "Phase 1: Thorough Review"
    expected_agents: 8
    agents: [compliance, bug-detection, security, performance, architecture, api-contracts, error-handling, test-coverage]

  - name: "Phase 2: Gaps Review"
    expected_agents: 4
    agents: [compliance, bug-detection, security, performance]

  - name: "Synthesis"
    expected_agents: 4
    agents: [synthesis (x4 with different category pairs)]
```

### Quick Review (7 invocations)

```yaml
phases:
  - name: "Review"
    expected_agents: 4
    agents: [bug-detection, security, error-handling, test-coverage]

  - name: "Synthesis"
    expected_agents: 3
    agents: [synthesis (x3 with different category pairs)]
```

## Integration

This tracking is maintained as internal state during workflow execution and converted to the Usage Summary output format (see `shared/output-format.md`) before generating the review output.
