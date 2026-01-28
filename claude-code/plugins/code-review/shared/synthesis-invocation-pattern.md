# Synthesis Agent Invocation Pattern

This document defines the invocation pattern for the synthesis-agent. The synthesis agent is designed to be invoked **multiple times in parallel** with different category pairs.

See `${CLAUDE_PLUGIN_ROOT}/agents/synthesis-agent.md` for the full agent definition and analysis logic.

## Invocation Parameters

When launching the synthesis agent, the orchestrating command MUST provide:

```yaml
# Required parameters for each synthesis agent invocation:
synthesis_input:
  category_a:
    name: "Security"                # First category to analyze
    findings: [...]                 # All findings from category_a (Phase 1 + Phase 2)
  category_b:
    name: "Performance"             # Second category to analyze
    findings: [...]                 # All findings from category_b (Phase 1 + Phase 2)
  cross_cutting_question: "Do any security fixes introduce performance issues?"
  files_content: [...]              # File diffs and full content for context
```

## Parallel Invocation Pattern

Commands launch 5 instances of this agent simultaneously, each with different category pairs.

**Authoritative source for category pairs:** See `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md` "Synthesis" sections for the definitive list of pairs and cross-cutting questions for both deep review (5 pairs) and quick review (3 pairs).

Each instance operates independently and returns its own `cross_cutting_insights` list. The orchestrating command merges all results.

## Invocation Format (Required)

When invoking this agent, use the following YAML structure in the prompt:

```yaml
# REQUIRED: Synthesis agent invocation format
synthesis_input:
  category_a:
    name: "Security"
    findings:
      - title: "SQL injection in getUser"
        file: "src/db/users.ts"
        line: 23
        severity: "Critical"
        description: "User input concatenated into SQL query"
        fix_type: "diff"
        fix_diff: |
          - const query = `SELECT * FROM users WHERE id = ${userId}`;
          + const query = 'SELECT * FROM users WHERE id = ?';
          + const result = await db.query(query, [userId]);

  category_b:
    name: "Performance"
    findings:
      - title: "N+1 query in user list"
        file: "src/services/users.ts"
        line: 45
        severity: "Major"
        description: "Database query inside forEach loop"
        fix_type: "prompt"
        fix_prompt: "Batch the user queries using WHERE IN clause"

  cross_cutting_question: "Do any security fixes introduce performance issues?"

  files_content:
    - path: "src/db/users.ts"
      diff: |
        @@ -20,6 +20,8 @@
        +  const query = `SELECT * FROM users WHERE id = ${userId}`;
        +  return await db.query(query);
      full_content: "[full file content here]"
```

## Example Task Tool Invocation

```
Task(
  subagent_type: "code-review:synthesis-agent",
  model: "sonnet",
  prompt: """
Analyze cross-cutting concerns between Security and Performance findings.

synthesis_input:
  category_a:
    name: "Security"
    findings:
      - title: "SQL injection in getUser"
        file: "src/db/users.ts"
        line: 23
        severity: "Critical"
        description: "User input concatenated into SQL query"
        fix_type: "diff"
        fix_diff: |
          - const query = `SELECT * FROM users WHERE id = ${userId}`;
          + const query = 'SELECT * FROM users WHERE id = ?';

  category_b:
    name: "Performance"
    findings:
      - title: "N+1 query in user list"
        file: "src/services/users.ts"
        line: 45
        severity: "Major"

  cross_cutting_question: "Do any security fixes introduce performance issues?"

  files_content:
    - path: "src/db/users.ts"
      diff: "[diff content]"
      full_content: "[full file]"

Follow the synthesis-agent instructions to identify cross-cutting concerns.
Return findings as cross_cutting_insights YAML list.
"""
)
```
