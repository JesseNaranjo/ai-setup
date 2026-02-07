# Validation Rules

This document defines the validation process for issues found by review agents.

## Contents

- [Batch Validation Process](#batch-validation-process)
  - [Grouping Strategy](#grouping-strategy)
  - [Validator Model Assignment](#validator-model-assignment)
  - [Quick Review Validation Scope](#quick-review-validation-scope)
  - [Cross-Cutting Insight Validation](#cross-cutting-insight-validation)
  - [Batch Validator Prompt](#batch-validator-prompt)
  - [Auto-Validation (Skip Validation)](#auto-validation-skip-validation)
  - [Auto-Validation Output](#auto-validation-output)
  - [Common False Positives to Check](#common-false-positives-to-check)
  - [Validation Output](#validation-output)
- [Aggregation Rules](#aggregation-rules)
  - [Remove Invalid Issues](#1-remove-invalid-issues)
  - [Apply Severity Downgrades](#2-apply-severity-downgrades)
  - [Deduplicate Issues](#3-deduplicate-issues)
  - [Consensus Detection Algorithm](#4-consensus-detection-algorithm)
- [Severity Classification](#severity-classification)
- [Do NOT Validate These (Skip Validation)](#do-not-validate-these-skip-validation)
- [False Positive Rules](#false-positive-rules)
  - [Do NOT Flag](#do-not-flag)
  - [Deep vs Quick Review Differences](#deep-vs-quick-review-differences)
- [Category-Specific False Positive Rules](#category-specific-false-positive-rules)

## Batch Validation Process

To optimize cost and latency, issues are validated in batches grouped by file rather than individually.

### Grouping Strategy

1. **Group by file**: All issues for the same file are validated in a single agent call
2. **Group by validator model**: Issues requiring same model (Opus vs Sonnet) are grouped together
3. **Benefit**: Reduces agent invocations from N issues to M files (typically 60-80% reduction)

### Validator Model Assignment

| Issue Category | Validator Model |
|----------------|-----------------|
| Accuracy | Opus |
| API Contracts | Sonnet |
| Architecture | Opus |
| Bugs | Opus |
| Clarity | Sonnet |
| Completeness | Opus |
| Compliance | Sonnet |
| Consistency | Sonnet |
| Error Handling | Sonnet |
| Examples | Opus |
| Performance | Opus |
| Security | Opus |
| Structure | Sonnet |
| Technical Debt | Sonnet |
| Test Coverage | Sonnet |

**Cross-cutting insights** (from synthesis-code-agent or synthesis-docs-agent) always use **Opus** for validation.

**Note for Quick Reviews:** Despite the quick review philosophy of using Sonnet where possible, cross-cutting insights still use Opus for validation because they represent novel connections between categories that require more nuanced judgment to validate. The time savings of Sonnet validation does not justify the risk of missing subtle cross-category interactions.

### Quick Review Validation Scope

Quick reviews validate **Critical and Major severity issues only**. Minor issues and Suggestions skip the validation phase entirely to optimize for speed. This means:

- Critical/Major issues: Full validation with appropriate model
- Minor issues: Auto-accepted without validation
- Suggestions: Auto-accepted without validation

This differs from deep reviews, which validate all severity levels.

### Cross-Cutting Insight Validation

Synthesis agents produce `cross_cutting_insights` that require special validation:

1. **Integration Point**: Cross-cutting insights are added to the issue pool AFTER synthesis completes, BEFORE validation begins
2. **Category Assignment**: Each insight specifies a `category` field - use this for validator model assignment
3. **Validator Model**: Cross-cutting insights always use **Opus** for validation
4. **Related Findings Check**: Validator must verify that BOTH `related_findings` references exist in the original findings

**Cross-Cutting Validation Prompt Extension**:

```yaml
# For cross-cutting insights, add to the standard validation prompt:
cross_cutting_context:
  related_finding_a:
    category: "Security"
    title: "SQL injection in getUser"
    file: "src/db/users.ts"
    line: 23
  related_finding_b:
    category: "Performance"
    title: "N+1 query in user list"
    file: "src/services/users.ts"
    line: 45

Additional validation checks for cross-cutting insights:
1. Verify both related findings are real issues (not false positives)
2. Verify the cross-cutting insight describes a genuine interaction between the two
3. Verify the insight is NOT a duplicate of either original finding
4. Verify the insight adds value beyond what each category found independently
```

**Cross-Cutting Deduplication**: If a cross-cutting insight duplicates an issue from the original category findings:
- Mark as INVALID with reason: "Duplicates existing finding from [category]"
- The original finding takes precedence

### Batch Validator Prompt

For each file with issues, launch ONE validator with all issues for that file:

```yaml
Validate these issues in {file_path}:

issues_to_validate:
  - id: 1
    title: "{title}"
    line: {line}
    category: "{category}"
    severity: "{severity}"
    description: "{description}"

  - id: 2
    title: "{title}"
    ...

Your task:
1. Read the file to verify each issue exists
2. For EACH issue, determine if it's a real problem or false positive
3. Verify severity classification is appropriate
4. Return verdict for each issue

Return format (YAML):
validations:
  - id: 1
    verdict: VALID|INVALID|DOWNGRADE
    new_severity: # only if DOWNGRADE
    reason: "brief explanation"
  - id: 2
    ...
```

### Auto-Validation (Skip Validation)

Some high-confidence patterns skip validation entirely and are marked `auto_validated: true`:

**Domain-specific patterns:** See `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules-code.md` (code reviews) or `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules-docs.md` (documentation reviews).

**Notes on pattern matching:**
- Patterns are case-insensitive for SQL keywords
- Patterns require at least one character (`[^'"]+`) to avoid matching empty string placeholders like `password = ""`
- Patterns check for common variable names indicating user input: `req`, `request`, `params`, `query`, `body`, `input`, `user`
- Empty catch pattern allows a single comment line to avoid flagging intentional empty catches with explanation

### Auto-Validation Output

```yaml
issues:
  - title: "Hardcoded database password"
    auto_validated: true
    confidence_pattern: "hardcoded_credential"
    ...
```

### Common False Positives to Check

Validators should check for these common false positive patterns:

For the complete list of issues that should NOT be flagged, see the "False Positive Rules" section below.

1. **Pre-existing issues**: The issue existed before the changes being reviewed
2. **Context makes it correct**: The code appears wrong but has valid context
3. **Handled elsewhere**: The issue is addressed in another part of the codebase
4. **Explicit ignore comments**: The issue is intentionally silenced (lint-ignore, etc.)
5. **Theoretical only**: The issue requires unrealistic conditions to manifest
6. **Test/dev code**: The issue is in test or development-only code paths
7. **Internal code**: The issue is in internal code with no external exposure

### Validation Output

Each validator returns:
- **VALID**: Issue is confirmed, keep original severity
- **INVALID**: Issue is a false positive, remove from results
- **DOWNGRADE**: Issue is real but severity should be lower

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

**Step 1: Group by file**
Collect all validated issues from all agents, grouped by file path.

**Step 2: Detect overlapping issues**
Two issues overlap if same file AND line ranges intersect:
- Issue A at line L1 (or range L1-L2, where L2 defaults to L1 if single line)
- Issue B at line M1 (or range M1-M2)
- Overlap: NOT (L2 < M1 OR M2 < L1)

**Step 3: Build clusters**
Group overlapping issues transitively (if A overlaps B and B overlaps C, cluster = {A, B, C}).

**Step 4: Assign badges and merge**
For each cluster with issues from N unique agents:
- **[2 agents]**: N = 2
- **[3+ agents]**: N >= 3
- No badge: N = 1

**Step 5: Merge cluster into single issue**
- Severity: highest in cluster
- Description: longest (most detailed)
- Categories: combined (e.g., "Security, Bugs")
- Badge: prepend to title

**Example:**
```
security-agent: "SQL injection" at users.ts:45-48 (Critical)
bug-detection-agent: "Unsanitized input" at users.ts:46 (Major)
→ Merged: "[2 agents] SQL injection" at users.ts:45-48 (Critical, Security+Bugs)
```

## Severity Classification

Reference `${CLAUDE_PLUGIN_ROOT}/shared/severity-definitions.md` for canonical severity definitions and action requirements.

## Do NOT Validate These (Skip Validation)

- Issues in test files (unless explicitly testing production code paths)
- Style suggestions without functional impact
- Documentation suggestions

## False Positive Rules

Issues that should NOT be flagged by review agents.

### Do NOT Flag

#### Correct Code

- Code that handles edge cases in non-obvious but valid ways
- Intentional patterns that look unusual but serve a purpose
- Something that appears to be a bug but is actually correct behavior

#### Linter Territory

- Formatting issues covered by prettier/eslint/other formatters
- Import ordering or unused import warnings
- Issues that a linter will catch (do not run the linter to verify)

#### Pedantic Concerns

- Minor formatting inconsistencies
- Nitpicks that a senior engineer would not flag
- Style preferences that don't affect functionality

#### Pre-existing Issues

- Issues in files with changes that existed before these changes
- Issues in the codebase that are not related to the current changes
- Pre-existing code patterns that weren't modified

#### Scope Limitations

- General code quality concerns unless explicitly required in AI Agent Instructions
- Issues in test code or development-only paths unless specifically reviewing tests
- Suggestions for features or patterns not in scope
- Theoretical edge cases that are extremely unlikely in practice

#### Silenced Issues

- Code with lint-disable comments for specific rules
- Intentionally suppressed warnings with documented reasons
- Issues mentioned in AI Agent Instructions but explicitly silenced in code

### Deep vs Quick Review Differences

#### Deep Review (More Thorough)

Deep review can flag more issues but should still avoid:
- Issues explicitly silenced in code
- Pre-existing issues unrelated to changes
- Pure style preferences

#### Quick Review (Stricter)

Quick review is optimized for speed and should be extra conservative:
- Focus only on blocking issues that must be fixed before merge
- Ignore minor style concerns
- Skip theoretical edge cases

## Category-Specific False Positive Rules

Each category has specific exclusions in addition to the general false positive rules above.

**Domain-specific rules:** See `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules-code.md` (code reviews) or `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules-docs.md` (documentation reviews).
