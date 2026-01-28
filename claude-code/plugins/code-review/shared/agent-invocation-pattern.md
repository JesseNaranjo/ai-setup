# Agent Invocation Pattern

This document defines how to invoke review agents via the Task tool.

## Related Files

- `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md` - Phase definitions and model selection table
- `${CLAUDE_PLUGIN_ROOT}/shared/agent-common-instructions.md` - Common agent instructions (MODE, false positives, gaps, pre-existing issue detection)
- `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md` - Authoritative output schema reference
- `${CLAUDE_PLUGIN_ROOT}/agents/*.md` - Individual agent definitions

## Subagent Types

Plugin agents are registered as subagent types with the pattern `code-review:[agent-name]`:

| Agent | Subagent Type |
|-------|---------------|
| API Contracts | `code-review:api-contracts-agent` |
| Architecture | `code-review:architecture-agent` |
| Bug Detection | `code-review:bug-detection-agent` |
| Compliance | `code-review:compliance-agent` |
| Error Handling | `code-review:error-handling-agent` |
| Performance | `code-review:performance-agent` |
| Security | `code-review:security-agent` |
| Synthesis | `code-review:synthesis-agent` |
| Technical Debt | `code-review:technical-debt-agent` |
| Test Coverage | `code-review:test-coverage-agent` |

## Invocation Template

Use the Task tool to launch each agent:

```
Task(
  subagent_type: "code-review:security-agent",  // Use registered agent type
  model: "opus",  // See orchestration-sequence.md for model selection
  description: "[Agent name] review for [scope]",
  prompt: """
MODE: thorough  // or gaps, quick

project_type: nodejs  // or dotnet, or both

files_to_review:
  - path: "src/services/OrderService.ts"
    has_changes: true
    tier: "critical"
    diff: |
      @@ -45,8 +45,12 @@
      +  const orders = await Order.findAll();
      +  for (const order of orders) {
      +    order.items = await OrderItem.findByOrderId(order.id);
      +  }
    full_content: |
      import { Order, OrderItem } from '../models';
      // ... full file content

  # Peripheral files (for staged reviews - unchanged files for context)
  - path: "src/db/schema.ts"
    has_changes: false
    tier: "peripheral"
    preview: |
      // Database schema definitions
      import { Entity, Column } from 'typeorm';
      // ... first 50 lines
    line_count: 1250
    full_content_available: true

ai_instructions:
  - source: "CLAUDE.md"
    content: |
      ## Security
      - All API endpoints MUST have authentication

related_tests:
  - path: "src/services/OrderService.test.ts"
    content: |
      describe('OrderService', () => { ... });

// For gaps mode only:
previous_findings:
  - title: "Issue already found in thorough mode"
    file: "src/services/OrderService.ts"
    line: 45
    range: "45-48"  # Line range if multi-line, null if single line
    category: "Performance"
    severity: "Critical"

// Skill-derived instructions (from --skills argument, orchestrator-interpreted):
skill_instructions:
  focus_areas:
    - "OWASP Top 10 vulnerabilities"
  checklist:
    - category: "Injection Vulnerabilities"
      severity: "Critical"
      items: ["SQL injection", "Command injection", "XSS"]
  auto_validate:
    - "hardcoded_password"
  false_positive_rules:
    - "Passwords in test files"
  methodology:
    approach: "Explore multiple interpretations before concluding"
    steps:
      - "Consider why code might be correct"
      - "Brainstorm failure modes"
    questions:
      - "What assumptions does this code make?"

// Additional instructions (from settings file body + --prompt argument):
additional_instructions: |
  # Project-Specific Instructions
  [content from .claude/code-review.local.md markdown body]

  # Command-Line Instructions
  [content from --prompt argument]

Return findings as YAML per agent examples in your agent file.
"""
)
```

## Model Selection

See `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md` for the authoritative model selection table.

**Important**: Always pass the `model` parameter explicitly when invoking Task:
- Use `model: "opus"` for bug-detection, performance, security, and technical-debt agents in thorough mode (require nuanced judgment)
- Use `model: "sonnet"` for compliance and all other agents in thorough mode (pattern-based detection)
- Use `model: "sonnet"` for gaps mode (constrained task with prior findings context)
- Use `model: "sonnet"` for synthesis agents (cross-category correlation)

**Note**: Gaps mode uses Sonnet because it receives prior findings context and follows explicit checklists, reducing the complexity of the task.

## Common Agent Input

See `${CLAUDE_PLUGIN_ROOT}/shared/agent-common-instructions.md` for standard agent inputs (files, project type, MODE, skill_instructions, previous_findings).

Each agent returns issues following the YAML schema defined in each agent file.
