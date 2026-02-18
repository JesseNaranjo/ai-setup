# Validation & Aggregation

## Batch Validation

Issues grouped by file, then by validator model. One validator per file. Quick reviews validate **Critical and Major only** (Minor/Suggestions skip).

**Cross-cutting insights:** Validated AFTER synthesis, BEFORE main validation. Always **Opus**. Checks: (1) Both `related_findings` references exist and aren't false positives, (2) genuine interaction between the two, (3) not a duplicate of either original, (4) adds value beyond individual findings. Duplicates of originals → INVALID.

**Quick reviews:** Cross-cutting insights still use Opus (novel cross-category connections require nuanced judgment).

**Skip validation for:** Issues in test files (unless testing production paths), style suggestions without functional impact, documentation suggestions.

### Validator Prompt Schema

| Field | Type | Notes |
|-------|------|-------|
| `file_path` | string | File being validated |
| `issues_to_validate[].id` | integer | Sequential issue ID |
| `issues_to_validate[].title` | string | Issue title |
| `issues_to_validate[].line` | integer | Line number |
| `issues_to_validate[].category` | string | Issue category |
| `issues_to_validate[].severity` | string | Issue severity |
| `issues_to_validate[].description` | string | Issue description |

Return format:
```yaml
validations:
  - id: 1
    verdict: VALID|INVALID|DOWNGRADE
    new_severity: # only if DOWNGRADE
    reason: "brief explanation"
```

### Auto-Validation

Issues matching auto-validation patterns (below) skip validation, marked `auto_validated: true`.

### Common FP Checks

- Explicit ignore comments (lint-ignore, suppress)
- Handled elsewhere in codebase
- Test/dev-only code paths
- Internal code with no external exposure

### Verdicts

**VALID** (confirmed, keep severity), **INVALID** (false positive, remove), **DOWNGRADE** (real but lower severity).

## Validator Model Assignment

Use agent `model` frontmatter for validation. Cross-cutting insights always **Opus**.

## Aggregation

1. **Remove Invalid**: Filter INVALID issues
2. **Apply Downgrades**: Update severity for DOWNGRADE verdicts
3. **Deduplicate**: Merge same-file overlapping-range issues; keep most detailed description; use highest severity
4. **Consensus Detection**: Group by file, detect overlapping issues (same file, intersecting ranges: NOT (L2 < M1 OR M2 < L1)). Build clusters transitively (A overlaps B, B overlaps C → {A,B,C}). Merge each: highest severity, longest description, combined categories, badge (`[2 agents]` or `[3+ agents]`)

## Auto-Validation Patterns (Documentation)

Issues matching these patterns skip validation entirely and are marked `auto_validated: true`:

**Accuracy patterns:**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `api_signature_mismatch` | N/A (detected via code comparison) | Documented function signature differs from implementation |
| `missing_parameter_doc` | N/A (detected via param comparison) | Parameter exists in code but not documented |
| `outdated_version_reference` | N/A (detected via version comparison) | Documentation references older version than package.json/csproj |

**Clarity patterns:**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `undefined_acronym` | `\b[A-Z]{2,}\b(?!.*\([^)]+\))` | Acronym used without expansion (first occurrence) |
| `passive_voice_instruction` | N/A (detected via NLP analysis) | Instructions written in passive voice |

**Completeness patterns:**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `missing_api_doc` | N/A (detected via export comparison) | Exported function/class has no documentation |
| `empty_section` | `^#+\s+.+\n\s*\n(?=^#+)` | Section header with no content before next header |
| `missing_error_handling_doc` | N/A (detected via throw analysis) | Function throws but no error documentation |

**Consistency patterns:**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `inconsistent_terminology` | N/A (detected via term frequency analysis) | Same concept named differently across docs |
| `inconsistent_heading_style` | N/A (detected via heading pattern analysis) | Mixed title case and sentence case headings |

**Examples patterns:**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `example_syntax_error` | N/A (detected via parser) | Code example has syntax error |
| `example_undefined_import` | N/A (detected via import analysis) | Example uses undefined import |

**Structure patterns:**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `broken_internal_link` | `\[([^\]]+)\]\((?!https?://)([^)]+)\)` | Internal markdown link (check target exists) |
| `missing_ai_instruction_header` | N/A (detected via header check) | CLAUDE.md missing required header comment |
| `ai_instruction_wrong_location` | N/A (detected via path check) | AI instruction file in wrong directory |

---

## Output Format

Authoritative output schema reference for all agents.

### Output Process

Generate: Header → Summary Table (reviewed categories only) → Issues (by severity) → Cross-Cutting Insights (if any) → Test Recommendations (if applicable). Display in terminal. Write to file (generated filename or `--output-file`). Print: "Review saved to: [filepath]"

### Filename

Pattern: `yyyy-mm-dd_few-words-summary_review-type.md` — 2-4 lowercase-hyphenated words from file names or "staged-changes". Review-type matches command name and depth. Under 80 chars. Example: `2026-02-05_project-docs_docs-review-deep.md`

### Review Header

`## Documentation Review` heading, then `**Reviewed:** [N] file(s) | **Branch:** [branch-name]` and `**Review Depth:** [review-depth-description]` on separate lines.

**No issues:** Header → "No issues found. All checks passed:" → checked categories → "Files reviewed:" → file list.

### Issues Found

Header followed by:
1. **Summary table**: Categories × severity (`Critical | Major | Minor | Suggestions`) + `**Total**` row. Only reviewed categories (6 deep, 4 quick).
2. **Severity sections**: `### Critical Issues (Must Fix)`, `### Major Issues (Should Fix)`, `### Minor Issues`, `### Suggestions` — only sections with issues.
3. **Cross-Cutting Insights** (after severity, before Test Recommendations)
4. **Test Recommendations** (if applicable)
5. Footer: `Review saved to: [filepath]`

### Cross-Cutting Insights

Only if synthesis agents produced `cross_cutting_insights`; omit section entirely if none. Format: `**[N]. [Insight title]** \`[Severity]\` \`[Primary Category] + [Secondary Category]\`` on first line, backtick `path/to/file.ts:line` on second, description referencing related findings from each category, then fix using diff or prompt format.

### Issue Entry Format

Format: `**[N]. [Title]** \`[Severity]\` \`[Category]\`` followed by optional consensus badge, then backtick `file:line` on next line, description paragraph, and `**Fix**:` with diff or prompt. Consensus badge: `[2 agents]` or `[3+ agents]` after category — omit if single agent.

### Actionable Fix Formats

**One entry per unique issue.** Fixes must be directly applicable.

**Inline diffs** (fix_type: diff): Single-location, ≤10 lines, complete drop-in replacement (no `...` or partial code), no changes elsewhere. Use standard diff code block with `-`/`+` lines.

**Fix prompts** (fix_type: prompt): Multi-location/structural fixes. Label `**Fix prompt** (copy to Claude Code):` then blockquote (`>`) with action verb, specific files and numbered changes.
