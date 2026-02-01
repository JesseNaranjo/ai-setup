# Output Format Details

This document contains detailed format templates and examples for code review output. It is loaded on-demand during the output generation phase when detailed formatting is needed.

**Parent Document:** `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md` - Core output generation process

## Contents

- [Usage Summary Section](#usage-summary-section)
  - [Format](#format)
  - [Review Type Values](#review-type-values)
  - [Status Indicators](#status-indicators)
  - [Duration Formatting](#duration-formatting)
  - [Quick Review Phase Names](#quick-review-phase-names)
  - [Agents Invoked Calculation](#agents-invoked-calculation)
  - [Handling Anomalies](#handling-anomalies)
  - [Partial/Interrupted Reviews](#partialinterrupted-reviews)
- [Review Header](#review-header)
  - [Review Depth Descriptions](#review-depth-descriptions)
- [No Issues Found](#no-issues-found)
  - [Deep Review (9 categories)](#deep-review-9-categories)
  - [Quick Review (4 categories + synthesis)](#quick-review-4-categories--synthesis)
- [Issue Entry Format](#issue-entry-format)
  - [Severity Badge](#severity-badge)
  - [Category Badge](#category-badge)
  - [Consensus Badge](#consensus-badge)
  - [File Location](#file-location)
- [Actionable Fix Formats](#actionable-fix-formats)
  - [Fix Type Classification](#fix-type-classification)
  - [Inline Diffs](#inline-diffs-fix_type-diff)
  - [Fix Prompts](#fix-prompts-fix_type-prompt)
  - [Legacy Format Support](#legacy-format-support)

---

## Usage Summary Section

The Usage Summary appears BEFORE the Code Review header to provide visibility into which agents were invoked and their timing.

### Format

```markdown
## Usage Summary

| Metric | Value |
|--------|-------|
| Review Type | Deep (19 invocations) |
| Total Duration | 3m 5s |
| Agents Invoked | 19 of 19 planned |

### Phase Breakdown

| Phase | Duration | Agents | Status |
|-------|----------|--------|--------|
| Phase 1: Thorough | 1m 44s | 9/9 | ✓ |
| Phase 2: Gaps | 44s | 5/5 | ✓ |
| Synthesis | 29s | 5/5 | ✓ |

<details>
<summary>Agent Timing Details</summary>

**Phase 1: Thorough Review** (1m 44s)
| Agent | Model | Duration | Findings | Status |
|-------|-------|----------|----------|--------|
| api-contracts-agent | sonnet | 31s | 0 | ✓ |
| architecture-agent | opus | 35s | 0 | ✓ |
| bug-detection-agent | opus | 1m 44s | 5 | ✓ |
| compliance-agent | sonnet | 27s | 2 | ✓ |
| error-handling-agent | sonnet | 28s | 2 | ✓ |
| performance-agent | opus | 58s | 1 | ✓ |
| security-agent | opus | 1m 12s | 3 | ✓ |
| technical-debt-agent | opus | 52s | 2 | ✓ |
| test-coverage-agent | sonnet | 33s | 4 | ✓ |

**Phase 2: Gaps Review** (44s)
| Agent | Model | Duration | Findings | Status |
|-------|-------|----------|----------|--------|
| bug-detection-agent | sonnet | 44s | 2 | ✓ |
| compliance-agent | sonnet | 22s | 1 | ✓ |
| performance-agent | sonnet | 41s | 1 | ✓ |
| security-agent | sonnet | 38s | 0 | ✓ |
| technical-debt-agent | sonnet | 36s | 1 | ✓ |

**Synthesis** (29s)
| Agent | Model | Duration | Findings | Status |
|-------|-------|----------|----------|--------|
| synthesis (Architecture+Test Coverage) | sonnet | 29s | 1 | ✓ |
| synthesis (Bugs+Error Handling) | sonnet | 22s | 0 | ✓ |
| synthesis (Compliance+Bugs) | sonnet | 27s | 0 | ✓ |
| synthesis (Compliance+Technical Debt) | sonnet | 24s | 0 | ✓ |
| synthesis (Performance+Security) | sonnet | 25s | 1 | ✓ |

</details>

---
```

### Review Type Values

| Command | Review Type Value |
|---------|-------------------|
| `/deep-code-review` | Deep (19 invocations) |
| `/deep-code-review-staged` | Deep (19 invocations) |
| `/deep-docs-review` | Deep Documentation (13 invocations) |
| `/quick-code-review` | Quick (7 invocations) |
| `/quick-code-review-staged` | Quick (7 invocations) |
| `/quick-docs-review` | Quick Documentation (7 invocations) |

### Status Indicators

| Indicator | Meaning |
|-----------|---------|
| `✓` | Completed successfully |
| `✗` | Failed |
| `-` | Skipped (via skip_agents setting) |
| `[!]` | Too fast - see `usage-tracking.md` "Timing Anomaly Detection" for thresholds |
| `[*]` | Too slow - see `usage-tracking.md` "Timing Anomaly Detection" for thresholds |

### Duration Formatting

Format durations for readability:
- Under 60 seconds: `27s`
- 60+ seconds: `1m 44s`
- 60+ minutes: `1h 5m 30s`

### Quick Review Phase Names

For quick reviews, use these phase names:
- "Review" (instead of "Phase 1: Thorough")
- "Synthesis"

### Agents Invoked Calculation

`Agents Invoked` = count of agents with status "completed" or "failed"
`planned` = total expected agents for the review type (excluding skipped)

Example with skip_agents:
- Deep review with `skip_agents: ["architecture"]`
- Expected: 15 planned (16 - 1 skipped)
- Display: "15 of 15 planned"

### Handling Anomalies

When timing anomalies are detected, append the indicator to the status:

```markdown
| bug-detection-agent | opus | 8s | 0 | ✓ [!] |
| security-agent | opus | 3m 45s | 7 | ✓ [*] |
```

### Partial/Interrupted Reviews

If a review is interrupted before completion:
- Show available data for completed agents
- Use `(incomplete)` for phases that didn't finish
- Example: "Agents Invoked: 10 of 16 planned (incomplete)"

---

## Review Header

All reviews start with this header:

```markdown
## Code Review

**Reviewed:** [N] file(s) | **Branch:** [branch-name]
**Review Depth:** [review-depth-description]
```

### Review Depth Descriptions

| Command | Review Depth Description |
|---------|-------------------------|
| `/deep-code-review` | Deep (19 invocations: 9 thorough + 5 gaps + 5 synthesis) |
| `/deep-code-review-staged` | Deep (19 invocations: 9 thorough + 5 gaps + 5 synthesis) |
| `/quick-code-review` | Quick (7 invocations: 4 review + 3 synthesis) |
| `/quick-code-review-staged` | Quick (7 invocations: 4 review + 3 synthesis) |

## No Issues Found

When no issues are found after validation, list only the categories that were actually checked.

### Deep Review (9 categories)

For deep reviews (`/deep-code-review`, `/deep-code-review-staged`), list all 9 categories:

```markdown
## Code Review

**Reviewed:** [N] file(s) | **Branch:** [branch-name]
**Review Depth:** Deep (19 invocations: 9 thorough + 5 gaps + 5 synthesis)

No issues found. All checks passed:
- API Contracts
- Architecture
- Bugs (logical errors, edge cases)
- Compliance (AI instructions)
- Error Handling
- Performance
- Security
- Technical Debt
- Test Coverage

Files reviewed:
- [file1.ts]
- [file2.ts]
```

### Quick Review (4 categories + synthesis)

For quick reviews (`/quick-code-review`, `/quick-code-review-staged`), list only the 4 categories that were checked:

```markdown
## Code Review

**Reviewed:** [N] file(s) | **Branch:** [branch-name]
**Review Depth:** Quick (7 invocations: 4 review + 3 synthesis)

No issues found. All checks passed:
- Bug detection
- Error handling
- Security analysis
- Test coverage
- Cross-cutting synthesis

Files reviewed:
- [file1.ts]
- [file2.ts]
```

---

## Issue Entry Format

Each issue entry follows this format:

```markdown
**[N]. [Issue title]** `[Severity]` `[Category]` [Consensus badge if applicable]
`path/to/file.ts:line-range`

[Description of the issue - what it is and why it matters]

[For small fixes - suggestion block]
[For larger fixes - prompt block]
```

### Severity Badge

Use inline code format:
- `` `Critical` ``
- `` `Major` ``
- `` `Minor` ``
- `` `Suggestion` ``

### Category Badge

Use inline code format:
- `` `API Contracts` ``
- `` `Architecture` ``
- `` `Bugs` ``
- `` `Compliance` ``
- `` `Error Handling` ``
- `` `Performance` ``
- `` `Security` ``
- `` `Technical Debt` ``
- `` `Test Coverage` ``

### Consensus Badge

When multiple agents flag the same issue:
- `[2 agents]` - Flagged by 2 agents
- `[3+ agents]` - Flagged by 3 or more agents

### File Location

Use inline code format with path and line range:
`` `src/utils/helper.ts:42-48` ``

---

## Actionable Fix Formats

Fixes should be actionable - users can apply them directly without interpretation. Choose the format based on fix complexity.

### Fix Type Classification

| Fix Type | When to Use | Format |
|----------|-------------|--------|
| `diff` | Single location, ≤10 lines, exact replacement known | Inline diff block |
| `prompt` | Multi-location, structural changes, or requires context decisions | Claude Code prompt |

### Inline Diffs (fix_type: diff)

For simple, single-location fixes where the exact code change is known:

````markdown
**Fix**:
```diff
- const user = db.query(`SELECT * FROM users WHERE id = ${id}`);
+ const user = db.query('SELECT * FROM users WHERE id = ?', [id]);
```
````

**Requirements for diff blocks:**
- Lines starting with `-` show code to remove
- Lines starting with `+` show code to add
- Must be complete (no "..." or partial code)
- Must be a drop-in replacement
- Must not require changes elsewhere
- Should be ≤10 lines changed

**Multi-line diff example:**
````markdown
**Fix**:
```diff
- async function getUser(id) {
-   const user = await db.query(`SELECT * FROM users WHERE id = ${id}`);
-   return user;
- }
+ async function getUser(id: string): Promise<User | null> {
+   const user = await db.query('SELECT * FROM users WHERE id = ?', [id]);
+   if (!user) return null;
+   return user;
+ }
```
````

### Fix Prompts (fix_type: prompt)

For complex fixes requiring multiple locations, structural changes, or context decisions:

````markdown
**Fix prompt** (copy to Claude Code):
> Refactor UserService to separate authentication logic into AuthService:
> 1. Create src/services/AuthService.ts
> 2. Move login(), logout(), validateToken() from UserService
> 3. Update imports in src/api/auth.ts and src/middleware/auth.ts
````

**Requirements for prompt blocks:**
- Use blockquote format (`>`) for easy copying
- Start with the action verb (Refactor, Extract, Add, Fix, etc.)
- List specific files and changes when multi-location
- Be specific enough that Claude Code can execute without clarification

**Single-location prompt example (when diff is impractical):**
````markdown
**Fix prompt** (copy to Claude Code):
> Add comprehensive input validation to processOrder in src/services/orders.ts:72-95. Validate orderId is a valid UUID, items array is non-empty with valid product IDs and quantities > 0, and customerId exists in the database. Throw descriptive errors for each validation failure.
````

### Legacy Format Support

For backward compatibility, `suggestion` blocks are still supported and equivalent to diffs:

````markdown
```suggestion
const result = await fetchData();
if (!result) {
  throw new Error('Failed to fetch data');
}
```
````

This is treated as `fix_type: diff` internally.
