# Code Review Validation & Aggregation

This section defines the validation process, aggregation rules, and domain-specific patterns for code review commands.

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

Some high-confidence patterns skip validation entirely and are marked `auto_validated: true`. See Auto-Validation Patterns (Code) section below.

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
| API Contracts | Sonnet |
| Architecture | Opus |
| Bugs | Opus |
| Compliance | Sonnet |
| Error Handling | Sonnet |
| Performance | Opus |
| Security | Opus |
| Technical Debt | Sonnet |
| Test Coverage | Sonnet |

**Cross-cutting insights** (from synthesis-code-agent) always use **Opus** for validation.

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

## Auto-Validation Patterns (Code)

Some high-confidence patterns skip validation entirely and are marked `auto_validated: true`:

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

**Compliance patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `missing_authorize_attribute` | `\[(?:Http(?:Get\|Post\|Put\|Delete\|Patch)\|Route)\][^[]*(?<!\[Authorize\])\s*public\s+(?:async\s+)?(?:Task<)?(?:IActionResult\|ActionResult)` | ASP.NET endpoint without [Authorize] attribute |
| `wrong_case_filename` | N/A (detected via filesystem comparison) | Filename violates project naming convention |
| `explicit_must_violation` | N/A (detected via rule matching) | Code violates a MUST rule from AI instructions |
| `missing_required_jsdoc` | `export\s+(?:async\s+)?function\s+\w+\s*\([^)]*\)\s*(?::\s*\w+)?\s*\{(?!\s*/\*\*)` | Exported function without JSDoc comment |

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

**Technical Debt patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `deprecated_npm_package` | N/A (detected via `npm ls` output) | npm package with deprecation warning |
| `todo_without_issue` | `(?:\/\/\|#)\s*TODO(?:\([^)]*\))?:?\s+(?!.*(?:#\d+\|ISSUE-\|JIRA-\|TICKET-))` | TODO/FIXME without issue reference |
| `commented_out_code` | `^(?:\s*(?:\/\/\|#).*\n){10,}` | 10+ consecutive lines of commented code |
| `hack_comment` | `(?:\/\/\|#\|\/\*)\s*(?:HACK\|WORKAROUND\|XXX)\s*[:\-]?` | Explicit HACK/WORKAROUND/XXX marker |
| `outdated_callback` | `function\s+\w+\s*\([^)]*,\s*(?:callback\|cb\|done)\s*\)` | Callback pattern in async context |

## Category-Specific False Positive Rules (Code)

Each category has specific exclusions in addition to the general false positive rules in `${CLAUDE_PLUGIN_ROOT}/shared/agent-common-instructions.md`.

- **API Contracts**: Additive-only changes; beta/experimental APIs marked as unstable; changes following established deprecation process
- **Architecture**: Pragmatic compromises with clear justification; patterns overkill for project scale
- **Bug Detection**: Code with explicit comments explaining why it's correct
- **Compliance**: Explicit override comments; ambiguous rules with reasonable compliance; style preferences not stated as rules
- **Error Handling**: Errors intentionally ignored with explicit comments; logging-only catch blocks as intended behavior
- **Performance**: Micro-optimizations without measurable impact; code that runs rarely
- **Security**: Internal-only code with no untrusted input exposure; vulnerabilities mitigated elsewhere in the code
- **Technical Debt**: Dependencies pinned for compatibility (documented reason); dead code that's conditionally compiled (build flags); TODO comments referencing issue tracking (TODO(#123)); workarounds with documented upstream bugs; class components in projects intentionally supporting older React versions
- **Test Coverage**: Code impractical to unit test (better for integration tests); code covered by higher-level tests; generated code/boilerplate; dead code that should be removed rather than tested
