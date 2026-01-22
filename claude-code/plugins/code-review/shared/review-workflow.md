# Code Review Workflow Orchestration

This document defines the orchestration logic for the code review workflow. Agent definitions, language configs, validation rules, and output formats are in separate files.

## Related Files

- `agents/*.md` - Individual agent definitions with MODE parameter
- `languages/nodejs.md` - Node.js/TypeScript specific checks
- `languages/dotnet.md` - .NET/C# specific checks
- `shared/validation-rules.md` - Validation process and rules
- `shared/output-format.md` - Output formatting templates
- `shared/severity-definitions.md` - Canonical severity definitions

## Severity Classification

Reference `shared/severity-definitions.md` for canonical severity definitions.

| Severity | Action Required |
|----------|-----------------|
| **Critical** | Must fix before merge |
| **Major** | Should fix before merge |
| **Minor** | Can merge, fix soon |
| **Suggestion** | Optional |

## Agent Configuration

### Available Agents

| Agent | File | Model | Supported Modes |
|-------|------|-------|-----------------|
| Compliance | `agents/compliance-agent.md` | Sonnet | thorough, gaps, quick |
| Bug Detection | `agents/bug-detection-agent.md` | Opus | thorough, gaps, quick |
| Security | `agents/security-agent.md` | Opus | thorough, gaps, quick |
| Performance | `agents/performance-agent.md` | Opus | thorough, gaps, quick |
| Architecture | `agents/architecture-agent.md` | Sonnet | thorough, quick |
| API Contracts | `agents/api-contracts-agent.md` | Sonnet | thorough, quick |
| Error Handling | `agents/error-handling-agent.md` | Sonnet | thorough, quick |
| Test Coverage | `agents/test-coverage-agent.md` | Sonnet | thorough, quick |

### MODE Parameter

Each agent accepts a MODE parameter:
- **thorough**: Comprehensive review, check all issues
- **gaps**: Focus on subtle issues that might be missed, receives prior findings context
- **quick**: Fast pass on critical issues only

## Orchestration Sequence

This section defines the authoritative execution order for review pipelines. Each step must complete before the next begins.

### Deep Review Orchestration (16 agent invocations)

1. **Steps 1-3: Input, Context, Content**
   - Validate input, discover context, gather file content
   - OUTPUT: Files to review, diffs, AI instructions, test files

2. **Phase 1: Thorough Review** (8 agents in parallel)
   - Launch: bug, security, performance (Opus) + compliance, architecture, api, error-handling, test-coverage (Sonnet)
   - MODE: `thorough` for all agents
   - WAIT: All 8 agents must complete before proceeding
   - OUTPUT: Phase 1 findings (grouped by category)

3. **Phase 2: Gaps Review** (4 Sonnet agents in parallel)
   - Launch: compliance, bug, security, performance
   - MODE: `gaps`
   - Model: Sonnet (cost-optimized for constrained task with prior findings context)
   - INPUT: Phase 1 findings passed as `previous_findings` (each agent receives only its own category)
   - WAIT: All 4 agents must complete before proceeding
   - OUTPUT: Phase 2 findings (subtle issues, edge cases)

4. **Synthesis** (4 agents in parallel)
   - Launch: 4 instances of synthesis-agent with different category pairs
   - INPUT: ALL findings from Phase 1 AND Phase 2
   - Pairs: Security+Performance, Architecture+Test Coverage, Bugs+Error Handling, Compliance+Bugs
   - WAIT: All 4 synthesis agents must complete before proceeding
   - OUTPUT: `cross_cutting_insights` list

5. **Validation**
   - INPUT: All issues from Phase 1 + Phase 2 + Synthesis
   - Process: Batch validation by file (see `validation-rules.md`)
   - OUTPUT: VALID/INVALID/DOWNGRADE verdict for each issue

6. **Aggregation**
   - Filter invalid issues, apply downgrades, deduplicate, add consensus badges
   - OUTPUT: Final issue list

7. **Output**
   - Generate Usage Summary from tracking data
   - Generate formatted report, write to file

### Quick Review Orchestration (7 agent invocations)

1. **Steps 1-3: Input, Context, Content**
   - Same as deep review
   - OUTPUT: Files to review, diffs, AI instructions, test files

2. **Review** (4 agents in parallel)
   - Launch: bug, security, error-handling, test-coverage
   - MODE: `quick` for all agents
   - WAIT: All 4 agents must complete before proceeding
   - OUTPUT: Quick review findings

3. **Synthesis** (3 agents in parallel)
   - Launch: 3 instances of synthesis-agent with different category pairs
   - INPUT: All findings from step 2
   - Pairs: Bugs+Error Handling, Security+Bugs, Bugs+Test Coverage
   - WAIT: All 3 synthesis agents must complete before proceeding
   - OUTPUT: `cross_cutting_insights` list

4. **Validation**
   - Critical/Major issues only (Minor and Suggestions skip validation)
   - OUTPUT: VALID/INVALID/DOWNGRADE verdict

5. **Aggregation & Output**
   - Same as deep review

## Usage Tracking Protocol

Throughout the review workflow, maintain usage tracking to record agent invocations and timing. See `shared/usage-tracking.md` for the complete schema.

### Initialization (Before Step 4)

Before launching the first agent:

1. Record `review_started_at` timestamp for the review
2. Initialize the phases array based on review type:
   - **Deep review**: 3 phases (Phase 1: 8 agents, Phase 2: 4 agents, Synthesis: 4 agents)
   - **Quick review**: 2 phases (Review: 4 agents, Synthesis: 3 agents)
3. Initialize each phase with an empty agents array

### Recording Each Agent Invocation

For every agent launch:

1. **Before Task tool call**: Record `agent_started_at` timestamp
2. **After Task tool returns**:
   - Record `agent_ended_at` timestamp
   - Calculate `agent_duration_seconds` = agent_ended_at - agent_started_at
   - Record `task_id` from Task tool return (confirms actual invocation)
   - Set `status` to "completed" (or "failed" if error)
3. **For skipped agents** (via skip_agents setting):
   - Set `status` to "skipped"
   - Set `agent_duration_seconds` to 0
   - No timestamps or task_id

### Confirming Actual Invocation

The Task tool returns a `task_id` when an agent completes. Record this ID as proof of actual invocation:
- Present: Agent was actually invoked
- Absent with duration: Agent may have failed

### Parallel Execution Timing

When agents run in parallel within a phase:
- **phase_started_at**: Timestamp when first agent launched
- **phase_ended_at**: Timestamp when last agent completed
- **phase_duration_seconds**: phase_ended_at - phase_started_at
- Individual agent durations are independent
- Sum of individual durations may exceed phase duration (expected with parallelism)

### Timing Anomaly Detection

Flag potential issues based on timing:

| Condition | Indicator | Concern |
|-----------|-----------|---------|
| Opus < 15s | `[!]` too fast | May have errored early |
| Sonnet < 10s | `[!]` too fast | May have errored early |
| Synthesis < 5s | `[!]` too fast | May have errored early |
| Opus > 180s | `[*]` too slow | May be stuck |
| Sonnet > 120s | `[*]` too slow | May be stuck |
| Synthesis > 90s | `[*]` too slow | May be stuck |

### Generating Usage Summary

At the end of the workflow (before generating review output):

1. Read from the usage_tracking structure maintained during workflow
2. Format according to `shared/output-format.md` Usage Summary section
3. Calculate phase totals and detect timing anomalies
4. Include agent details in expandable section

## Review Configurations

### Deep Review (deep-review, deep-review-staged)

Deep review uses a **two-phase sequential approach** for Opus agents to reduce duplicates and improve gaps analysis:

**Phase 1: Thorough Review (8 agents in parallel)**

| Agent | Model | MODE |
|-------|-------|------|
| compliance-agent | Sonnet | thorough |
| bug-detection-agent | Opus | thorough |
| security-agent | Opus | thorough |
| performance-agent | Opus | thorough |
| architecture-agent | Sonnet | thorough |
| api-contracts-agent | Sonnet | thorough |
| error-handling-agent | Sonnet | thorough |
| test-coverage-agent | Sonnet | thorough |

**Phase 2: Gaps Review with Context (4 Sonnet agents in parallel)**

After Phase 1 completes, pass thorough findings to gaps mode agents:

| Agent | Model | MODE | Context |
|-------|-------|------|---------|
| compliance-agent | Sonnet | gaps | Phase 1 compliance findings |
| bug-detection-agent | Sonnet | gaps | Phase 1 bug findings |
| security-agent | Sonnet | gaps | Phase 1 security findings |
| performance-agent | Sonnet | gaps | Phase 1 performance findings |

**Note**: Sonnet is used for gaps mode because this phase is constrained by prior findings context and explicit checklists in `gaps-mode-rules.md`, making it suitable for a more cost-efficient model while retaining quality. Phase 1 thorough mode catches most issues; gaps mode focuses on subtle issues that complement those findings.

Gaps mode agents receive prior findings to:
- Skip issues already flagged (same file + overlapping lines)
- Focus on subtle issues that might be missed
- Find edge cases and boundary conditions

Total: 16 agent invocations (8 Phase 1 + 4 Phase 2 + 4 Synthesis), executed in three sequential phases to enable context passing.

### Quick Review (quick-review, quick-review-staged)

Launch 4 agent invocations in parallel:

| Agent | MODE |
|-------|------|
| bug-detection-agent | quick |
| security-agent | quick |
| error-handling-agent | quick |
| test-coverage-agent | quick |

Quick review also includes **cross-agent synthesis** with 3 synthesis agents:

| Input Categories | Cross-Cutting Question |
|-----------------|------------------------|
| Bugs + Error Handling | "Do identified bugs have proper error handling in fix paths?" |
| Security + Bugs | "Do security issues introduce or relate to bugs?" |
| Bugs + Test Coverage | "Are identified bugs covered by tests?" |

Total: 7 agent invocations (4 review + 3 synthesis).

## Settings Application

Settings from `.claude/code-review.local.md` affect the workflow at specific points. This section documents how each setting is processed.

### skip_agents (Agent Filtering)

**Applied in:** Step 4 (Review Execution), before launching agents

**Implementation:**
```
Before launching each agent:
1. Get the agent name (e.g., "compliance", "security", "performance")
2. Check if agent name is in skip_agents list
3. If yes: Skip this agent, do not launch it
4. If no: Launch agent as normal
```

**Example:**
```yaml
skip_agents: ["architecture", "api-contracts"]
```

With this setting, deep review launches only 6 Phase 1 agents (instead of 8), reducing total invocations.

**Adjustment to counts:**
- Deep review: `(8 - skipped_count)` Phase 1 + `4` Phase 2 + `4` Synthesis
- Quick review: `(4 - skipped_count)` Review + `3` Synthesis

### min_severity (Severity Filtering)

**Applied in:** Step 6 (Aggregation), after validation

**Implementation:**
```
After validation completes:
1. Parse min_severity setting (default: "suggestion")
2. Define severity order: critical > major > minor > suggestion
3. For each validated issue:
   - If issue.severity < min_severity: Remove from output
   - If issue.severity >= min_severity: Keep in output
```

**Severity comparison:**
| min_severity | Included Severities |
|--------------|---------------------|
| `suggestion` | All (critical, major, minor, suggestion) |
| `minor` | critical, major, minor |
| `major` | critical, major |
| `critical` | critical only |

**Example:**
```yaml
min_severity: "major"
```

This filters out Minor and Suggestion issues from the final report.

### additional_test_patterns (Test Pattern Merging)

**Applied in:** Step 2 (Context Discovery), when finding related test files

**Implementation:**
```
When finding related test files:
1. Load default test patterns from languages/*.md
2. Get additional_test_patterns from settings (default: [])
3. Merge patterns: combined = default_patterns + additional_test_patterns
4. Use combined patterns for test file discovery
```

**Example:**
```yaml
additional_test_patterns:
  - "**/*.integration.ts"
  - "**/*.e2e.ts"
```

These patterns are merged with default patterns like `*.test.ts`, `*.spec.ts`.

### custom_rules (Custom Rule Passing)

**Applied in:** Step 4 (Review Execution), when launching compliance-agent

**Implementation:**
```
When launching compliance-agent:
1. Get custom_rules from settings (default: [])
2. If custom_rules is not empty:
   - Add "Custom Rules" section to agent prompt
   - Include each rule with pattern, message, and severity
3. Agent checks these rules in addition to CLAUDE.md rules
```

**Agent prompt addition:**
```yaml
# Include in compliance-agent prompt:
custom_rules:
  - pattern: "console\\.log"
    message: "Remove console.log statements before committing"
    severity: "minor"
```

**Example settings:**
```yaml
custom_rules:
  - pattern: "\\bTODO\\b"
    message: "Resolve TODO comments before merging"
    severity: "suggestion"
```

### Project Instructions (Markdown Body)

**Applied in:** Step 4 (Review Execution), to all agents

**Implementation:**
```
After loading settings file:
1. Extract markdown body (content after YAML frontmatter)
2. If body is not empty:
   - Add to all agent prompts as "Project-Specific Instructions"
   - This provides context about the project's conventions and exceptions
```

**Agent prompt addition:**
```
project_instructions: |
  # Project Context
  This is a high-security financial application...
```

## Workflow Steps

### Step 1: Input Validation

Verify:
- Current directory is a git repository
- Files/changes exist to review
- Parse command arguments

### Step 2: Context Discovery

1. Find all AI Agent Instructions files:
   - `CLAUDE.md` files (root and directories)
   - `.ai/AI-AGENT-INSTRUCTIONS.md` files
   - `.github/copilot-instructions.md` files

2. Detect project type and language per file:

   **Per-File Language Detection:**
   For each file being reviewed, detect its language:

   - **By extension**:
     - `.ts`, `.tsx`, `.js`, `.jsx`, `.mjs`, `.cjs` → Node.js/TypeScript
     - `.cs` → .NET/C#
     - `.py` → Python (future support)

   - **By nearest project file**:
     - Walk up directories to find `package.json` or `*.csproj`
     - Use that project's language config

   - **For monorepos/mixed codebases**:
     - Apply language-specific checks PER FILE
     - Group files by language when reporting to agents
     - Example context: "Node.js files: [a.ts, b.ts], .NET files: [c.cs]"

   **Language Override:**
   Commands accept `--language` parameter to force detection:
   - `--language nodejs` - Force Node.js/TypeScript checks for all files
   - `--language dotnet` - Force .NET/C# checks for all files

3. Find related test files based on detected project type(s)

### Step 3: Content Gathering

1. Get current branch name
2. For files with changes: get diff AND full file content
3. For files without changes: get full file content
4. Read related test files for context

### Step 4: Review Execution

Review execution varies by review type:

#### Deep Review: Two-Phase Sequential Approach

**Phase 1: Launch 8 agents with thorough mode in parallel**
- 3 Opus agents (bug-detection, security, performance) in thorough mode
- 5 Sonnet agents (compliance, architecture, api-contracts, error-handling, test-coverage) in thorough mode

Wait for all Phase 1 agents to complete and collect their findings.

**Phase 2: Launch 4 Sonnet agents with gaps mode in parallel**

Pass Phase 1 findings as context to gaps mode agents. Each agent receives findings from its own category:

```yaml
previous_findings:
  - title: "SQL injection in getUser"
    file: "src/db/users.ts"
    line: 23
    category: "Security"
    severity: "Critical"
    # Gaps agent should skip this - already flagged
```

Gaps mode agents use this context to:
- Skip issues that match prior findings (same file + overlapping line ranges)
- Focus analysis on areas not yet covered
- Find subtle issues that complement thorough findings

#### Quick Review: Single-Phase Parallel

Launch 4 agents with quick mode in parallel. No gaps phase.

#### Agent Invocation Pattern

Use the Task tool to launch each agent. Plugin agents are registered as subagent types with the pattern `code-review:[agent-name]`:

| Agent | Subagent Type |
|-------|---------------|
| Security | `code-review:security-agent` |
| Bug Detection | `code-review:bug-detection-agent` |
| Performance | `code-review:performance-agent` |
| Compliance | `code-review:compliance-agent` |
| Architecture | `code-review:architecture-agent` |
| API Contracts | `code-review:api-contracts-agent` |
| Error Handling | `code-review:error-handling-agent` |
| Test Coverage | `code-review:test-coverage-agent` |
| Synthesis | `code-review:synthesis-agent` |

Here is the exact invocation pattern:

```
Task(
  subagent_type: "code-review:security-agent",  // Use registered agent type
  model: "opus",  // See model selection table below
  description: "[Agent name] review for [scope]",
  prompt: """
MODE: thorough  // or gaps, quick

project_type: nodejs  // or dotnet, or both

files_to_review:
  - path: "src/services/OrderService.ts"
    has_changes: true
    diff: |
      @@ -45,8 +45,12 @@
      +  const orders = await Order.findAll();
      +  for (const order of orders) {
      +    order.items = await OrderItem.findByOrderId(order.id);
      +  }
    full_content: |
      import { Order, OrderItem } from '../models';
      // ... full file content

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
    category: "Performance"
    severity: "Critical"

Return findings as YAML per shared/output-schema-base.md.
"""
)
```

**Model Selection per Agent (Authoritative Source):**

This table is the **single source of truth** for agent-to-model mapping. Commands reference this table rather than duplicating it.

| Agent | Model (thorough) | Model (gaps) |
|-------|------------------|--------------|
| compliance-agent | sonnet | sonnet |
| bug-detection-agent | opus | sonnet |
| security-agent | opus | sonnet |
| performance-agent | opus | sonnet |
| architecture-agent | sonnet | N/A |
| api-contracts-agent | sonnet | N/A |
| error-handling-agent | sonnet | N/A |
| test-coverage-agent | sonnet | N/A |
| synthesis-agent | sonnet | N/A |

**Important**: Always pass the `model` parameter explicitly when invoking Task:
- Use `model: "opus"` for bug-detection, security, and performance agents in thorough mode (require nuanced judgment)
- Use `model: "sonnet"` for compliance and all other agents in thorough mode (pattern-based detection)
- Use `model: "sonnet"` for gaps mode (constrained task with prior findings context)
- Use `model: "sonnet"` for synthesis agents (cross-category correlation)

**Note**: Gaps mode uses Sonnet because it receives prior findings context and follows explicit checklists, reducing the complexity of the task.

#### Common Agent Input

Each agent receives:
- Current branch name
- Files to review (diffs and/or full content)
- Detected project type(s) and language per file
- Relevant AI Agent Instructions files
- Related test files
- MODE parameter
- Previous findings (gaps mode only, from same category)

Each agent returns issues following the YAML schema defined in each agent file.

#### Pre-Existing Issue Detection (For Staged/Diff Reviews)

**CRITICAL**: When reviewing staged changes or diffs, agents must only flag issues in CHANGED lines.

**Context provided to agents:**
1. Diff content with line markers (lines starting with `+` are additions)
2. Surrounding unchanged lines (for understanding context only)
3. Full file content (for reference only, not for flagging issues)

**Rules for what to flag:**
- ✅ Issue is in a line starting with `+` in the diff (newly added code)
- ✅ Change INTRODUCES the issue (e.g., removes null check that protected existing code)
- ✅ Change WORSENS an existing issue (e.g., increases scope of vulnerability)

**Do NOT flag:**
- ❌ Issues in unchanged code (lines without `+` prefix)
- ❌ Pre-existing problems not made worse by the change
- ❌ Style issues in untouched code nearby
- ❌ Issues visible in "full file" context but not in the diff

**Example:**
```diff
  function getUser(id) {
+   const user = await db.query(`SELECT * FROM users WHERE id = ${id}`);  // FLAG: SQL injection
    if (existingBuggyCode) {  // DO NOT FLAG: pre-existing, not in diff
      return null;
    }
+   return user;
  }
```

#### Automatic Cross-File Analysis

Agents automatically perform cross-file analysis when the reviewed code suggests cross-cutting concerns. This is triggered by:

- **Import/export statements** → Check for consumers, circular dependencies
- **Class/interface definitions** → Check implementations, inheritance chains
- **API contracts** → Check for breaking changes affecting consumers
- **Shared types/utilities** → Check all usages for consistency

When triggered, agents use Grep and Glob tools to discover related files, then Read to analyze relationships. This enables detection of:
- Unused exports
- Circular dependencies
- Broken call chains (signature changed, callers not updated)
- Interface/implementation mismatches

See `agents/architecture-agent.md` for detailed cross-file analysis triggers and process.

### Synthesis Phase: Cross-Agent Synthesis

After the review phase and before validation, launch **synthesis agents** that analyze findings across categories to identify cross-cutting concerns.

**Purpose**: Catch issues that span multiple review categories, such as:
- Security fixes that introduce performance regressions
- Architectural changes lacking test coverage
- Bug fixes that break API contracts or lack error handling

#### Deep Review Synthesis (4 agents in parallel)

| Agent | Input Categories | Cross-Cutting Question |
|-------|-----------------|------------------------|
| `agents/synthesis-agent.md` | Security + Performance | "Do any security fixes introduce performance issues?" |
| `agents/synthesis-agent.md` | Architecture + Test Coverage | "Are architectural changes covered by tests?" |
| `agents/synthesis-agent.md` | Bugs + Error Handling | "Do identified bugs have proper error handling in fix paths?" |
| `agents/synthesis-agent.md` | Compliance + Bugs | "Do compliance violations introduce or mask bugs?" |

#### Quick Review Synthesis (3 agents in parallel)

| Agent | Input Categories | Cross-Cutting Question |
|-------|-----------------|------------------------|
| `agents/synthesis-agent.md` | Bugs + Error Handling | "Do identified bugs have proper error handling in fix paths?" |
| `agents/synthesis-agent.md` | Security + Bugs | "Do security issues introduce or relate to bugs?" |
| `agents/synthesis-agent.md` | Bugs + Test Coverage | "Are identified bugs covered by tests?" |

Each synthesis agent receives:
- All findings from both input categories
- The file diffs/content being reviewed
- Context about what each category found

**Synthesis Agent Output**:
```yaml
cross_cutting_insights:
  - title: "SQL injection fix uses unbounded query"
    related_findings:
      - security: "SQL injection in getUser"
      - performance: "Unbounded query in user lookup"
    # Both categories required - if only one has a finding, don't flag as cross-cutting
    insight: "The parameterized query fix doesn't limit result set size, potential DoS"
    category: "Performance"
    severity: "Major"
    file: "src/db/users.ts"
    line: 25
```

Synthesis insights are added to the issue pool before validation.

### Step 5: Validation

For each issue from the review phases, launch a validation subagent.

See `shared/validation-rules.md` for:
- Validator model assignment
- Validation prompt template
- Common false positive checks

### Step 6: Aggregation

1. Remove issues marked INVALID
2. Apply severity downgrades
3. Deduplicate by file+line range
4. Add consensus badges for multi-agent issues

See `shared/validation-rules.md` for full aggregation rules.

### Step 7: Generate Output

Format results according to `shared/output-format.md`.

### Step 8: Write Output

1. Display formatted review in terminal
2. Write to output file (default or specified via --output-file)
3. Print "Review saved to: [filepath]"

## Language-Specific Focus

Agents apply language-specific checks based on per-file detection:

- **Node.js/TypeScript files** (`.ts`, `.tsx`, `.js`, `.jsx`): See `languages/nodejs.md`
- **.NET/C# files** (`.cs`): See `languages/dotnet.md`

For mixed codebases (monorepos):
- Each file is analyzed with its own language config
- Agents receive file groupings by language
- Cross-language issues (e.g., API contract mismatches) are handled by architecture and API agents

## False Positive Guidelines

Do NOT flag these (across all agents):

- Pre-existing issues not introduced in the changes being reviewed
- Code that appears problematic but is actually correct in context
- Pedantic nitpicks that a senior engineer would not flag
- Issues that a linter/type checker will catch
- General quality concerns not explicitly required by AI Agent Instructions
- Issues with explicit ignore comments (lint-disable, etc.)
- Theoretical issues that require specific conditions to manifest
- Subjective style preferences not mandated by guidelines

## Notes

- Use git CLI for repository interactions (not GitHub CLI)
- Always cite file paths and line numbers for each issue
- Quote exact AI Agent Instructions rules when citing violations
- File paths should be relative to repository root
- Line numbers reference lines in working copy (not diff line numbers)
- Maintain a todo list to track review progress
- Each issue should appear only once in the output (deduplicate)
