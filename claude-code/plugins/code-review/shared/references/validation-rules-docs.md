# Documentation Review Validation & Aggregation

This section defines the validation process, aggregation rules, and domain-specific patterns for documentation review commands.

## Batch Validation Process

To optimize cost and latency, issues are validated in batches grouped by file rather than individually.

### Grouping Strategy

1. **Group by file**: All issues for the same file are validated in a single agent call
2. **Group by validator model**: Issues requiring same model (Opus vs Sonnet) are grouped together

### Quick Review Validation Scope

Quick reviews validate **Critical and Major severity issues only**. Minor issues and Suggestions skip the validation phase entirely to optimize for speed. This means:

- Critical/Major issues: Full validation with appropriate model
- Minor issues: Auto-accepted without validation
- Suggestions: Auto-accepted without validation

This differs from deep reviews, which validate all severity levels.

### Cross-Cutting Insight Validation

Cross-cutting insights are added to the issue pool AFTER synthesis, BEFORE validation. Each specifies a `category` field for model assignment. Always validate with **Opus**.

**Validation checks:** (1) Both `related_findings` references exist and are not false positives, (2) describes genuine interaction between the two, (3) not a duplicate of either original finding, (4) adds value beyond individual category findings.

**Deduplication**: If a cross-cutting insight duplicates an original finding, mark INVALID ("Duplicates existing finding from [category]"). Original takes precedence.

### Batch Validator Prompt

**Skip validation for:** Issues in test files (unless explicitly testing production code paths), style suggestions without functional impact, and documentation suggestions.

For each file with issues, launch ONE validator. Prompt schema:

| Field | Type | Notes |
|-------|------|-------|
| `file_path` | string | File being validated |
| `issues_to_validate[].id` | integer | Sequential issue ID |
| `issues_to_validate[].title` | string | Issue title |
| `issues_to_validate[].line` | integer | Line number |
| `issues_to_validate[].category` | string | Issue category |
| `issues_to_validate[].severity` | string | Issue severity |
| `issues_to_validate[].description` | string | Issue description |

**Required return format:**

```yaml
validations:
  - id: 1
    verdict: VALID|INVALID|DOWNGRADE
    new_severity: # only if DOWNGRADE
    reason: "brief explanation"
```

### Auto-Validation (Skip Validation)

Some high-confidence patterns skip validation entirely and are marked `auto_validated: true`. See Auto-Validation Patterns (Documentation) section below.

### Common False Positives to Check

Validators should check for these pipeline-specific false positive patterns:

1. **Explicit ignore comments**: The issue is intentionally silenced (lint-ignore, suppress, etc.)
2. **Handled elsewhere**: The issue is addressed in another part of the codebase
3. **Test/dev code**: The issue is in test or development-only code paths
4. **Internal code**: The issue is in internal code with no external exposure

### Validation Output

Each validator returns: **VALID** (confirmed, keep severity), **INVALID** (false positive, remove), or **DOWNGRADE** (real but lower severity).

## Validator Model Assignment

| Issue Category | Validator Model |
|----------------|-----------------|
| Accuracy | Opus |
| Clarity | Sonnet |
| Completeness | Opus |
| Consistency | Sonnet |
| Examples | Opus |
| Structure | Sonnet |

**Cross-cutting insights** (from synthesis-docs-agent) always use **Opus** for validation.

**Quick Reviews:** Cross-cutting insights still use Opus for validation (novel cross-category connections require nuanced judgment).

## Aggregation Rules

After validation, perform these aggregation steps:

### 1. Remove Invalid Issues

Filter out all issues marked INVALID by their validator.

### 2. Apply Severity Downgrades

For issues marked DOWNGRADE:
- Update the severity to the new level specified by the validator
- Keep the original description and details

### 3. Deduplicate Issues

When multiple agents flag the same issue:
- Merge issues for the same file and overlapping line ranges
- Keep the most detailed description
- Use the highest severity among the merged issues
- Add a consensus badge indicating multiple agents flagged it

### 4. Consensus Detection Algorithm

1. **Group by file** and detect overlapping issues (same file, intersecting line ranges: NOT (L2 < M1 OR M2 < L1))
2. **Build clusters** transitively (if A overlaps B and B overlaps C, cluster = {A, B, C})
3. **Merge each cluster**: use highest severity, longest description, combined categories, and add badge (`[2 agents]` or `[3+ agents]`) prepended to title

## Auto-Validation Patterns (Documentation)

Some high-confidence patterns skip validation entirely and are marked `auto_validated: true`:

**Accuracy patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `api_signature_mismatch` | N/A (detected via code comparison) | Documented function signature differs from implementation |
| `missing_parameter_doc` | N/A (detected via param comparison) | Parameter exists in code but not documented |
| `outdated_version_reference` | N/A (detected via version comparison) | Documentation references older version than package.json/csproj |

**Clarity patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `undefined_acronym` | `\b[A-Z]{2,}\b(?!.*\([^)]+\))` | Acronym used without expansion (first occurrence) |
| `passive_voice_instruction` | N/A (detected via NLP analysis) | Instructions written in passive voice |

**Completeness patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `missing_api_doc` | N/A (detected via export comparison) | Exported function/class has no documentation |
| `empty_section` | `^#+\s+.+\n\s*\n(?=^#+)` | Section header with no content before next header |
| `missing_error_handling_doc` | N/A (detected via throw analysis) | Function throws but no error documentation |

**Consistency patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `inconsistent_terminology` | N/A (detected via term frequency analysis) | Same concept named differently across docs |
| `inconsistent_heading_style` | N/A (detected via heading pattern analysis) | Mixed title case and sentence case headings |

**Examples patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `example_syntax_error` | N/A (detected via parser) | Code example has syntax error |
| `example_undefined_import` | N/A (detected via import analysis) | Example uses undefined import |

**Structure patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `broken_internal_link` | `\[([^\]]+)\]\((?!https?://)([^)]+)\)` | Internal markdown link (check target exists) |
| `missing_ai_instruction_header` | N/A (detected via header check) | CLAUDE.md missing required header comment |
| `ai_instruction_wrong_location` | N/A (detected via path check) | AI instruction file in wrong directory |

