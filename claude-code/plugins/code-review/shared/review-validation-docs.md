# Validation & Aggregation

## Batch Validation

Group by file, then validator model. One validator per file. Quick: validate Critical/Major only.

**Cross-cutting insights:** Validate AFTER synthesis, BEFORE main. Always Opus. Checks: (1) both `related_findings` exist and aren't FPs, (2) genuine interaction, (3) not duplicate of either original, (4) adds value beyond individuals. Duplicates → INVALID. Quick reviews: still Opus.

**Skip validation:** Test file issues (unless testing production paths), style suggestions without functional impact, documentation suggestions.

### Validator Prompt Schema

`file_path` (string), `issues_to_validate[]`: `id` (int, sequential), `title`, `line` (int), `category`, `severity`, `description`, `context` (string, 5 lines surrounding the issue line) (all strings).

Return: `validations[]`: `id`, `verdict` (VALID/INVALID/DOWNGRADE), `new_severity` (DOWNGRADE only), `reason`.
Use `context` to verify the issue: confirm the described problem is plausible at the cited line. INVALID if context clearly contradicts the description.

### Auto-Validation

Issues matching patterns below skip validation, marked `auto_validated: true`.

### Common FP Checks

Ignore comments (lint-ignore, suppress). Handled elsewhere in codebase. Test/dev-only paths. Internal code with no external exposure.

### Verdicts

**VALID** (confirmed), **INVALID** (false positive, remove), **DOWNGRADE** (real but lower severity).

## Validator Model Assignment

Use agent `model` frontmatter. Cross-cutting insights always **Opus**.

## Aggregation

1. **Remove Invalid**: Filter INVALID issues
2. **Apply Downgrades**: Update severity for DOWNGRADE verdicts
3. **Deduplicate**: Merge same-file overlapping-range issues; keep most detailed, highest severity
4. **Consensus**: Group by file, detect overlapping (intersecting: NOT (L2 < M1 OR M2 < L1)). Cluster transitively (A-B, B-C → {A,B,C}). Merge: highest severity, longest description, combined categories, badge `[2 agents]`/`[3+ agents]`

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

**Process:** Header → Summary Table → Issues (by severity) → Cross-Cutting Insights → Test Recommendations. Display in terminal + write to file (generated or `--output-file`). Print: "Review saved to: [filepath]"

**Filename:** `yyyy-mm-dd_few-words-summary_review-type.md` — 2-4 lowercase-hyphenated words from file names or "staged-changes". Under 80 chars. Example: `2026-02-05_project-docs_docs-review-deep.md`

**Header:** `## Documentation Review`, then `**Reviewed:** [N] file(s) | **Branch:** [branch-name]` and `**Review Depth:** [description]`. **No issues:** Header → "No issues found. All checks passed:" → categories → "Files reviewed:" → file list.

**Issues:** (1) Summary table: Categories x severity (Critical|Major|Minor|Suggestions) + Total row, reviewed categories only (6 deep, 4 quick). (2) Severity sections: `### Critical Issues (Must Fix)`, `### Major Issues (Should Fix)`, `### Minor Issues`, `### Suggestions` — only if issues exist. (3) Cross-Cutting Insights. (4) Test Recommendations. (5) Footer.

**Cross-Cutting Insights:** Omit if none. Format: `**[N]. [Title]** \`[Severity]\` \`[Primary Category] + [Secondary Category]\``, backtick `file:line`, description referencing related findings, then fix (diff/prompt).

**Issue Entry:** `**[N]. [Title]** \`[Severity]\` \`[Category]\`` + optional badge (`[2 agents]`/`[3+ agents]`), backtick `file:line`, description, `**Fix**:` with diff or prompt.

**Fix Formats:** One per unique issue. **Diff** (fix_type: diff): single-location, ≤10 lines, complete drop-in (no `...`), standard diff block with `-`/`+`. **Prompt** (fix_type: prompt): multi-location/structural. Label `**Fix prompt** (copy to Claude Code):` then blockquote with action verb, files, numbered changes.
