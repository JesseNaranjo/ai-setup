---
name: deep-review-staged
allowed-tools: Task, Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git branch:*), Bash(git rev-parse:*), Bash(ls:*), Read, Write, Glob
description: Deep code review with 16 agent invocations (8 thorough + 4 gaps + 4 synthesis)
argument-hint: "[--output-file <path>] [--language nodejs|dotnet]"
---

Provide a deep 9-agent code review for staged git changes. This uses all 9 agents with both thorough and gaps modes for maximum coverage.

**Use this when**: You need the most comprehensive review before merging critical changes.

Example usage:
- `/deep-review-staged`
- `/deep-review-staged --output-file review.md`
- `/deep-review-staged --language nodejs` (force Node.js/TypeScript checks)
- `/deep-review-staged --language dotnet` (force .NET/C# checks)

---

## Step 0: Load Settings

See `${CLAUDE_PLUGIN_ROOT}/shared/settings-loader.md` for the settings loading process.

Check for `.claude/code-review.local.md` in the project root:
- If found and `enabled: false`, stop with message
- Apply settings: `output_dir`, `skip_agents`, `min_severity`, `language`, `custom_rules`
- Read project-specific instructions from markdown body

---

## Step 1: Staged Changes Validation

See `${CLAUDE_PLUGIN_ROOT}/shared/input-validation-staged.md` for the validation process.

**Default output file**: `{output_dir}/.deep-review-staged.md` (default: `.deep-review-staged.md`)

---

## Step 2: Context Discovery

See `${CLAUDE_PLUGIN_ROOT}/shared/context-discovery.md` for:
- AI Agent Instructions file patterns
- Project type detection rules
- Related test file patterns per language

---

## Step 3: Content Gathering

See `${CLAUDE_PLUGIN_ROOT}/shared/content-gathering-staged.md` for the content gathering process.

---

## Step 4: Two-Phase Deep Review

Deep review uses a sequential two-phase approach to reduce duplicates and improve gaps analysis.

### Phase 1: Thorough Review (8 agents in parallel)

Launch all agents with **thorough** mode:

| Agent | Model | MODE | Focus |
|-------|-------|------|-------|
| `${CLAUDE_PLUGIN_ROOT}/agents/compliance-agent.md` | Opus | thorough | Standards adherence |
| `${CLAUDE_PLUGIN_ROOT}/agents/bug-detection-agent.md` | Opus | thorough | Logical errors, null refs, off-by-one |
| `${CLAUDE_PLUGIN_ROOT}/agents/security-agent.md` | Opus | thorough | Injection, auth, secrets, OWASP |
| `${CLAUDE_PLUGIN_ROOT}/agents/performance-agent.md` | Opus | thorough | Complexity, memory, hot paths, N+1 |
| `${CLAUDE_PLUGIN_ROOT}/agents/architecture-agent.md` | Sonnet | thorough | Coupling, patterns, SOLID |
| `${CLAUDE_PLUGIN_ROOT}/agents/api-contracts-agent.md` | Sonnet | thorough | Breaking changes, compatibility |
| `${CLAUDE_PLUGIN_ROOT}/agents/error-handling-agent.md` | Sonnet | thorough | Try/catch gaps, resilience |
| `${CLAUDE_PLUGIN_ROOT}/agents/test-coverage-agent.md` | Sonnet | thorough | Missing tests, test suggestions |

Each agent receives:
- The current branch name
- A summary of the staged changes
- The staged diff
- Full file content for broader context
- The detected project type (for language-specific focus from `${CLAUDE_PLUGIN_ROOT}/languages/*.md`)
- The AI Agent Instructions files relevant to each staged file
- Related test files for context
- MODE parameter: **thorough**

**Wait for all Phase 1 agents to complete before proceeding.**

### Phase 2: Gaps Review with Context (4 Opus agents in parallel)

After Phase 1 completes, launch Opus agents with **gaps** mode, passing Phase 1 findings:

| Agent | Model | MODE | Prior Findings Context |
|-------|-------|------|------------------------|
| `${CLAUDE_PLUGIN_ROOT}/agents/compliance-agent.md` | Opus | gaps | Phase 1 compliance issues |
| `${CLAUDE_PLUGIN_ROOT}/agents/bug-detection-agent.md` | Opus | gaps | Phase 1 bug issues |
| `${CLAUDE_PLUGIN_ROOT}/agents/security-agent.md` | Opus | gaps | Phase 1 security issues |
| `${CLAUDE_PLUGIN_ROOT}/agents/performance-agent.md` | Opus | gaps | Phase 1 performance issues |

Each gaps agent receives all Phase 1 inputs PLUS:
- **previous_findings**: List of issues already flagged in this category from Phase 1

See `${CLAUDE_PLUGIN_ROOT}/shared/gaps-mode-rules.md` for gaps mode operation.

**Note**: Phase 2 only uses the 4 Opus agents because Sonnet agents only support thorough and quick modes.

---

## Step 5: Cross-Agent Synthesis

After Phase 1 and Phase 2 complete, launch 4 synthesis agents in parallel.

See `${CLAUDE_PLUGIN_ROOT}/agents/synthesis-agent.md` for full agent definition.

| Synthesis Agent | Input Categories | Cross-Cutting Question |
|-----------------|-----------------|------------------------|
| `${CLAUDE_PLUGIN_ROOT}/agents/synthesis-agent.md` | Security + Performance | "Do any security fixes introduce performance issues?" |
| `${CLAUDE_PLUGIN_ROOT}/agents/synthesis-agent.md` | Architecture + Test Coverage | "Are architectural changes covered by tests?" |
| `${CLAUDE_PLUGIN_ROOT}/agents/synthesis-agent.md` | Bugs + Error Handling | "Do identified bugs have proper error handling in fix paths?" |
| `${CLAUDE_PLUGIN_ROOT}/agents/synthesis-agent.md` | Compliance + Bugs | "Do compliance violations introduce or mask bugs?" |

Launch all 4 synthesis agents in parallel, each with their respective category pairs and findings from Phase 1 and Phase 2.

**Output**: `cross_cutting_insights` list added to the issue pool before validation.

---

## Step 6: Universal Validation

See `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md` for complete validation process including:
- Auto-validation patterns
- Batch grouping by file
- Validator model assignment

---

## Step 7: Aggregation

See `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md` for aggregation rules:
- Filter invalid issues
- Apply severity downgrades
- Deduplicate by file+line range
- Add consensus badges

---

## Step 8: Generate Output

See `${CLAUDE_PLUGIN_ROOT}/shared/output-generation.md` and `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md` for formatting.

**Review depth**: "Deep (9-agent analysis with thorough + gaps modes)"

**Categories to include in summary table**: All 8 (Compliance, Bugs, Security, Performance, Architecture, API Contracts, Error Handling, Test Coverage)

---

## Step 9: Write Output

See `${CLAUDE_PLUGIN_ROOT}/shared/output-generation.md` for write process.

**Default output file**: `.deep-review-staged.md`

---

## False Positives

See `${CLAUDE_PLUGIN_ROOT}/shared/false-positives.md` for issues that should NOT be flagged.

---

## Notes

- Use git CLI to interact with the repository. Do not use GitHub CLI.
- Create a todo list before starting.
- Cite each issue with file path and line numbers (e.g., `src/utils.ts:42-48`).
- When referencing AI Agent Instructions rules, quote the exact rule being violated.
- File paths should be relative to the repository root.
- Line numbers should reference the lines in the working copy (not diff line numbers).
