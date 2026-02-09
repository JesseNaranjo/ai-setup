# Code Review Validation Rules

This section defines the validation process and domain-specific patterns for code review commands.

## Batch Validation Process

See `${CLAUDE_PLUGIN_ROOT}/shared/validation-aggregation.md` for the common validation process (grouping, cross-cutting insight validation, batch validator prompt, false positives, verdicts).

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

### Auto-Validation (Skip Validation)

Some high-confidence patterns skip validation entirely and are marked `auto_validated: true`:

**Domain-specific patterns:** See Auto-Validation Patterns (Code) section below.

## Aggregation Rules

See `${CLAUDE_PLUGIN_ROOT}/shared/validation-aggregation.md` for aggregation rules (remove invalid, apply downgrades, deduplicate, consensus detection).

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
