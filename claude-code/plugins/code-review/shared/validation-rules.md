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

**Cross-cutting insights** (from synthesis-agent) always use **Opus** for validation.

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

**Accuracy patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `api_signature_mismatch` | N/A (detected via code comparison) | Documented function signature differs from implementation |
| `missing_parameter_doc` | N/A (detected via param comparison) | Parameter exists in code but not documented |
| `outdated_version_reference` | N/A (detected via version comparison) | Documentation references older version than package.json/csproj |

**Architecture patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `circular_dependency` | N/A (detected via import graph analysis) | Circular import/dependency between modules |
| `god_class_500_lines` | N/A (detected via line count) | Class/module exceeds 500 lines |
| `function_10_params` | `function\s+\w+\s*\([^)]*,\s*[^)]*,\s*[^)]*,\s*[^)]*,\s*[^)]*,\s*[^)]*,\s*[^)]*,\s*[^)]*,\s*[^)]*,\s*[^)]*\)` | Function has 10+ parameters |
| `direct_new_instantiation` | `(?:=\s*new\s+\w+Service\|=\s*new\s+\w+Repository)\s*\(` | Direct instantiation of service/repository dependency |

**Bug patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `empty_catch_block` | `catch\s*\([^)]*\)\s*\{\s*(?:\/\/[^\n]*)?\s*\}` | Empty catch block (allows single comment) |
| `missing_await` | `(?:const\|let\|var)\s+\w+\s*=\s*(?!await)[^;]*\basync\s+\w+\(` | Async function call without await |
| `null_dereference` | `(?:\?\.\s*\w+\s*\(\)\s*\.)\|(?:\w+\s*&&\s*\w+\.\w+\s*\?\s*\w+\.\w+\.\w+)` | Null access after optional chain or guard |

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

**Compliance patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `missing_authorize_attribute` | `\[(?:Http(?:Get\|Post\|Put\|Delete\|Patch)\|Route)\][^[]*(?<!\[Authorize\])\s*public\s+(?:async\s+)?(?:Task<)?(?:IActionResult\|ActionResult)` | ASP.NET endpoint without [Authorize] attribute |
| `wrong_case_filename` | N/A (detected via filesystem comparison) | Filename violates project naming convention |
| `explicit_must_violation` | N/A (detected via rule matching) | Code violates a MUST rule from AI instructions |
| `missing_required_jsdoc` | `export\s+(?:async\s+)?function\s+\w+\s*\([^)]*\)\s*(?::\s*\w+)?\s*\{(?!\s*/\*\*)` | Exported function without JSDoc comment |

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

**Performance patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `nested_loop_includes` | `for\s*\([^)]*\)[\s\S]*?for\s*\([^)]*\)[\s\S]*?\.(?:includes\|indexOf)\s*\(` | O(n²) nested loop with includes/indexOf |
| `select_star_in_loop` | `(?:for\|while\|forEach)[\s\S]*?SELECT\s+\*` | SELECT * inside a loop |
| `n_plus_one_query` | `(?:for\|while\|forEach\|\.map\|\.forEach)[\s\S]{0,200}(?:findOne\|findById\|query\|execute\|fetch)` | Database query inside iteration (N+1) |

**Security patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `hardcoded_password` | `(?:password\|passwd\|pwd)\s*[:=]\s*['"][^'"]+['"]` | Hardcoded password in assignment |
| `hardcoded_api_key` | `(?:api[_-]?key\|apikey)\s*[:=]\s*['"][^'"]+['"]` | Hardcoded API key |
| `hardcoded_token` | `(?:token\|bearer\|auth[_-]?token\|access[_-]?token)\s*[:=]\s*['"][^'"]+['"]` | Hardcoded auth token |
| `hardcoded_secret` | `(?:secret\|client[_-]?secret\|private[_-]?key)\s*[:=]\s*['"][^'"]+['"]` | Hardcoded secret/private key |
| `hardcoded_credentials` | `(?:credentials\|connection[_-]?string)\s*[:=]\s*['"][^'"]+['"]` | Hardcoded credentials object |
| `sql_injection_concat` | `(?:SELECT\|INSERT\|UPDATE\|DELETE\|FROM\|WHERE).*[+]\s*(?:req\|request\|params\|query\|body\|input\|user)` | SQL with string concatenation of user input |
| `sql_injection_template` | `(?:SELECT\|INSERT\|UPDATE\|DELETE\|FROM\|WHERE).*\$\{.*(?:req\|request\|params\|query\|body\|input\|user)` | SQL with template literal interpolation of user input |
| `eval_untrusted` | `eval\s*\(\s*(?:req\|request\|params\|query\|body\|input\|user)` | eval() with untrusted input |
| `new_function_untrusted` | `new\s+Function\s*\(\s*(?:req\|request\|params\|query\|body\|input\|user)` | new Function() with untrusted input |

**Structure patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `broken_internal_link` | `\[([^\]]+)\]\((?!https?://)([^)]+)\)` | Internal markdown link (check target exists) |
| `missing_ai_instruction_header` | N/A (detected via header check) | CLAUDE.md missing required header comment |
| `ai_instruction_wrong_location` | N/A (detected via path check) | AI instruction file in wrong directory |

**Technical Debt patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `deprecated_npm_package` | N/A (detected via `npm ls` output) | npm package with deprecation warning |
| `todo_without_issue` | `(?:\/\/\|#)\s*TODO(?:\([^)]*\))?:?\s+(?!.*(?:#\d+\|ISSUE-\|JIRA-\|TICKET-))` | TODO/FIXME without issue reference |
| `commented_out_code` | `^(?:\s*(?:\/\/\|#).*\n){10,}` | 10+ consecutive lines of commented code |
| `hack_comment` | `(?:\/\/\|#\|\/\*)\s*(?:HACK\|WORKAROUND\|XXX)\s*[:\-]?` | Explicit HACK/WORKAROUND/XXX marker |
| `outdated_callback` | `function\s+\w+\s*\([^)]*,\s*(?:callback\|cb\|done)\s*\)` | Callback pattern in async context |

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

### API Contracts

- Internal/private API changes
- Changes to APIs with no external consumers
- Additive changes that don't break existing consumers
- Changes that follow established deprecation process
- Beta/experimental APIs clearly marked as unstable

### Architecture

- Pragmatic compromises with clear justification
- Patterns that are overkill for the scale of the project
- Architecture decisions already documented and justified
- Temporary code with clear TODOs

### Bug Detection

- Code that appears buggy but is correct in context
- Defensive code that handles edge cases (unless it has a bug)
- Code with explicit comments explaining why it's correct

### Compliance

- Code that appears to violate a rule but has an explicit override comment
- Ambiguous rules where the code could reasonably be compliant
- Rules that don't apply to this file type or context
- Style preferences not explicitly stated as rules

### Error Handling

- Code where errors are intentionally ignored with explicit comments
- Errors that are handled at a higher level
- Internal code with documented error handling strategy
- Logging-only catch blocks where that's the intended behavior

### Performance

- Micro-optimizations that won't have measurable impact
- Performance issues in code that runs rarely
- Code that prioritizes readability over minor performance gains

### Security

- Internal-only code with no untrusted input exposure
- Code with explicit security comments explaining the design
- Vulnerabilities already mitigated elsewhere in the code

### Technical Debt

- Dependencies intentionally pinned for compatibility (documented reason)
- Legacy patterns in legacy modules explicitly marked as deprecated
- Dead code that's actually conditionally compiled (build flags)
- TODO comments that reference issue tracking (TODO(#123))
- Workarounds with documented upstream bugs and tracking
- Class components in projects supporting older React versions intentionally

### Test Coverage

- Private/internal implementation details
- Code that's impractical to unit test (better suited for integration tests)
- Code already covered by higher-level tests
- Test files themselves (don't require tests of tests)
- Generated code or boilerplate
- Configuration files or constants
- Dead code that should be removed rather than tested

### Accuracy (Documentation)

- Intentionally simplified examples (marked as "simplified" or "basic example")
- Pseudocode clearly marked as illustrative
- Version-specific documentation with version clearly noted
- Documentation for planned/upcoming features marked as such

### Clarity (Documentation)

- Jargon appropriate for stated expert audience
- Acronyms defined earlier in the same document
- Industry-standard terms in domain-specific docs (e.g., "REST" in API docs)
- Intentionally terse reference documentation (vs tutorials)
- Code comments within code blocks (different standards apply)

### Completeness (Documentation)

- Internal/private APIs not intended for external use
- Features clearly marked as experimental/unstable
- Configuration options with sensible defaults that rarely need changing
- Platform-specific docs when project only targets one platform
- Sections that would duplicate content available elsewhere (with link)

### Consistency (Documentation)

- Intentional variations for emphasis or clarity
- Code/API names that must match implementation (even if inconsistent with prose style)
- Quoted text that preserves original formatting
- Version-specific sections that intentionally differ
- External content (quotes, references) with different style

### Examples (Documentation)

- Pseudocode clearly marked as illustrative (not runnable)
- Intentionally simplified examples with explicit notes about what's omitted
- Partial examples with "..." indicating omitted code
- Examples for older versions in clearly versioned documentation
- Examples showing error cases (intentionally incorrect code to demonstrate what not to do)
- Shell examples with placeholder values like `<your-token>`

### Structure (Documentation)

- Intentionally orphaned archive/historical documents
- External links to known-stable resources (official docs, RFCs)
- Heading hierarchy violations in code-generated documentation
- Alternative navigation paths that are intentional (multiple entry points)
- AI instruction files in projects that don't use AI assistants (if explicitly stated)
