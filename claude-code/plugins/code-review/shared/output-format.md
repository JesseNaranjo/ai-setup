# Output Format

This document defines the output format for code review results and serves as the **authoritative output schema reference** for all agents. This file is loaded once during output generation phase.

## Contents

- [Related Files](#related-files)
- [Output Generation Process](#output-generation-process)
  - [Generate Usage Summary](#0-generate-usage-summary)
  - [Generate Review Output](#1-generate-review-output)
  - [Display Output](#2-display-output)
  - [Write to File](#3-write-to-file)
  - [Confirm Output](#4-confirm-output)
  - [Fix Formatting Rules](#fix-formatting-rules)
- [Issues Found](#issues-found)
- [Cross-Cutting Insights Section](#cross-cutting-insights-section)
- [File Output](#file-output)
- [Complete Output Example](#complete-output-example)

## Related Files

- `${CLAUDE_PLUGIN_ROOT}/shared/severity-definitions.md` - Canonical severity definitions
- `${CLAUDE_PLUGIN_ROOT}/shared/usage-tracking.md` - Usage tracking schema and timing anomaly thresholds
- `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md` - Validation and aggregation rules
- `${CLAUDE_PLUGIN_ROOT}/shared/references/complete-output-example.md` - Complete output example (reference)

## Output Generation Process

### 0. Generate Usage Summary

Before generating the Code Review output, generate the Usage Summary section:

1. **Read tracking data**: Access the usage_tracking structure maintained during workflow execution
2. **Calculate totals**:
   - Total duration = review ended_at - review started_at
   - Agents invoked = count of agents with status "completed" or "failed"
   - Planned agents = total expected minus skipped agents
3. **Calculate phase metrics**:
   - Phase duration = phase ended_at - phase started_at
   - Agents completed = count per phase
   - Total findings = sum of findings_count per agent
4. **Record findings_count**: For each agent, count the number of issues in their output and record as `findings_count`
5. **Handle missing data**: If any agent tracking data is missing (no task_id or timing), mark that agent's status as "unknown" with duration "??s" rather than omitting it. This ensures all expected agents appear in the output.
6. **Detect timing anomalies**: Apply thresholds from `${CLAUDE_PLUGIN_ROOT}/shared/usage-tracking.md` "Timing Anomaly Detection" section
7. **Format output**: Follow the Usage Summary Section format (loaded by command-common-steps.md)

**Output order:**
1. Usage Summary (this step)
2. Code Review header and content (next step)

See `${CLAUDE_PLUGIN_ROOT}/shared/usage-tracking.md` for the tracking schema.

### 1. Generate Review Output

1. **Header**: Use the review depth description provided by the command
2. **Summary Table**: Include only the categories that were reviewed
3. **Issues**: Group by severity (Critical, Major, Minor, Suggestions)
4. **Cross-Cutting Insights**: Include if synthesis agents produced insights
5. **Test Recommendations**: Include if applicable

### 2. Display Output

Display the formatted review in the terminal.

### 3. Write to File

Write the same content to a file:

| Command | Default Output File |
|---------|---------------------|
| `/deep-code-review` | `.deep-code-review.md` |
| `/deep-code-review-staged` | `.deep-code-review-staged.md` |
| `/deep-docs-review` | `.deep-docs-review.md` |
| `/quick-code-review` | `.quick-code-review.md` |
| `/quick-code-review-staged` | `.quick-code-review-staged.md` |
| `/quick-docs-review` | `.quick-docs-review.md` |

If `--output-file <path>` argument was provided, use that path instead.

### 4. Confirm Output

At the end, print: "Review saved to: [filepath]"

### Fix Formatting Rules

- **fix_type: diff** for single-location fixes ≤10 lines
  - Show inline diff block with `-` and `+` lines
  - Must be complete drop-in replacements (no "..." or partial code)

- **fix_type: prompt** for multi-location, structural, or complex fixes
  - Show copyable Claude Code prompt in blockquote format
  - Must be specific and actionable

**Only report ONE entry per unique issue. Do not duplicate issues.**

---

## Issues Found

When issues are found:

```markdown
## Code Review

**Reviewed:** [N] file(s) | **Branch:** [branch-name]
**Review Depth:** [review-depth-description]

### Summary

| Category | Critical | Major | Minor | Suggestions |
|----------|----------|-------|-------|-------------|
| API Contracts | 0 | 0 | 0 | 0 |
| Architecture | 0 | 0 | 0 | 0 |
| Bugs | 0 | 0 | 0 | 0 |
| Compliance | 0 | 0 | 0 | 0 |
| Error Handling | 0 | 0 | 0 | 0 |
| Performance | 0 | 0 | 0 | 0 |
| Security | 0 | 0 | 0 | 0 |
| Technical Debt | 0 | 0 | 0 | 0 |
| Test Coverage | 0 | 0 | 0 | 0 |
| **Total** | **0** | **0** | **0** | **0** |

### Critical Issues (Must Fix)

[List critical issues or "None"]

### Major Issues (Should Fix)

[List major issues or "None"]

### Minor Issues

[List minor issues or "None"]

### Suggestions

[List suggestions or "None"]

### Cross-Cutting Insights

[Issues identified by synthesis agents that span multiple categories. Show only if synthesis phase produced insights.]

### Test Recommendations

[Aggregated test case suggestions]

---
Review saved to: [filepath]
```

## Cross-Cutting Insights Section

After the regular severity-grouped issues, include cross-cutting insights from the synthesis phase. These are issues that span multiple review categories and represent ripple effects or hidden interactions.

### Format

```markdown
### Cross-Cutting Insights

Issues spanning multiple categories:

**[N]. [Insight title]** `[Severity]` `[Primary Category] + [Secondary Category]`
`path/to/file.ts:line`

[Description of the cross-cutting concern - what it is and why it matters. Reference which findings from each category are related.]

[Fix suggestion using diff or prompt format]
```

### Example Cross-Cutting Insight

```markdown
### Cross-Cutting Insights

Issues spanning multiple categories:

**10. Security Fix Creates Performance Regression** `Major` `Security + Performance`
`src/services/AuthService.cs:45-60`

The SQL parameterization fix (Issue #1) adds query preparation overhead that will be called on every request. The auth endpoint handles 1000+ requests/sec.

**Fix prompt** (copy to Claude Code):
> Cache the parameterized query preparation in AuthService.cs to avoid repeated compilation overhead. Use a static prepared statement or query cache with appropriate TTL.

**11. Architectural Change Missing Test Coverage** `Major` `Architecture + Test Coverage`
`src/interfaces/IUserService.ts:1-25`

The new IUserService interface (from architectural refactoring) has no corresponding tests. Existing tests still use the concrete class.

**Fix prompt** (copy to Claude Code):
> Add interface contract tests for IUserService in tests/interfaces/IUserService.test.ts. Ensure all implementations satisfy the interface contract.
```

### When to Include

- Only include if synthesis agents produced `cross_cutting_insights`
- Cross-cutting insights appear AFTER the regular severity-grouped issues
- Cross-cutting insights appear BEFORE Test Recommendations
- If no cross-cutting insights, omit this section entirely

## File Output

Write the review to a file:
- Default for `/deep-code-review`: `.deep-code-review.md`
- Default for `/deep-code-review-staged`: `.deep-code-review-staged.md`
- Default for `/deep-docs-review`: `.deep-docs-review.md`
- Default for `/quick-code-review`: `.quick-code-review.md`
- Default for `/quick-code-review-staged`: `.quick-code-review-staged.md`
- Default for `/quick-docs-review`: `.quick-docs-review.md`
- Custom path if `--output-file` argument provided

End with:
```markdown
---
Review saved to: [filepath]
```

## Complete Output Example

For a complete example showing all format elements together, see:
`${CLAUDE_PLUGIN_ROOT}/shared/references/complete-output-example.md`
