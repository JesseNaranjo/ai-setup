# Validation & Aggregation

## Batch Validation

Group by file, then validator model. One validator per file. Quick: validate Critical/Major only.

**Cross-cutting insights:** Validate AFTER synthesis, BEFORE main. Always Opus. Checks: (1) both `related_findings` exist and aren't FPs, (2) genuine interaction, (3) not duplicate of either original, (4) adds value beyond individuals. Duplicates → INVALID. Quick reviews: still Opus.

**Skip validation:** Test file issues (unless testing production paths), style suggestions without functional impact, documentation suggestions.

### Validator Prompt Schema

`file_path` (string), `issues_to_validate[]`: `id` (int, sequential), `title`, `line` (int), `category`, `severity`, `description` (all strings).

Return: `validations[]`: `id`, `verdict` (VALID/INVALID/DOWNGRADE), `new_severity` (DOWNGRADE only), `reason`.

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

## Auto-Validation Patterns (Code)

Issues matching these patterns skip validation entirely and are marked `auto_validated: true`:

**Architecture patterns:**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `circular_dependency` | N/A (detected via import graph analysis) | Circular import/dependency between modules |
| `god_class_500_lines` | N/A (detected via line count) | Class/module exceeds 500 lines |
| `function_10_params` | `function\s+\w+\s*\([^)]*,\s*[^)]*,\s*[^)]*,\s*[^)]*,\s*[^)]*,\s*[^)]*,\s*[^)]*,\s*[^)]*,\s*[^)]*,\s*[^)]*\)` | Function has 10+ parameters |
| `direct_new_instantiation` | `(?:=\s*new\s+\w+Service\|=\s*new\s+\w+Repository)\s*\(` | Direct instantiation of service/repository dependency |

**Bug patterns:**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `empty_catch_block` | `catch\s*\([^)]*\)\s*\{\s*(?:\/\/[^\n]*)?\s*\}` | Empty catch block (allows single comment) |
| `missing_await` | `(?:const\|let\|var)\s+\w+\s*=\s*(?!await)[^;]*\basync\s+\w+\(` | Async function call without await |
| `null_dereference` | `(?:\?\.\s*\w+\s*\(\)\s*\.)\|(?:\w+\s*&&\s*\w+\.\w+\s*\?\s*\w+\.\w+\.\w+)` | Null access after optional chain or guard |

**Compliance patterns:**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `missing_authorize_attribute` | `\[(?:Http(?:Get\|Post\|Put\|Delete\|Patch)\|Route)\][^[]*(?<!\[Authorize\])\s*public\s+(?:async\s+)?(?:Task<)?(?:IActionResult\|ActionResult)` | ASP.NET endpoint without [Authorize] attribute |
| `wrong_case_filename` | N/A (detected via filesystem comparison) | Filename violates project naming convention |
| `explicit_must_violation` | N/A (detected via rule matching) | Code violates a MUST rule from AI instructions |
| `missing_required_jsdoc` | `export\s+(?:async\s+)?function\s+\w+\s*\([^)]*\)\s*(?::\s*\w+)?\s*\{(?!\s*/\*\*)` | Exported function without JSDoc comment |

**Performance patterns:**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `nested_loop_includes` | `for\s*\([^)]*\)[\s\S]*?for\s*\([^)]*\)[\s\S]*?\.(?:includes\|indexOf)\s*\(` | O(n²) nested loop with includes/indexOf |
| `select_star_in_loop` | `(?:for\|while\|forEach)[\s\S]*?SELECT\s+\*` | SELECT * inside a loop |
| `n_plus_one_query` | `(?:for\|while\|forEach\|\.map\|\.forEach)[\s\S]{0,200}(?:findOne\|findById\|query\|execute\|fetch)` | Database query inside iteration (N+1) |

**Security patterns:**

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

**Technical Debt patterns:**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `deprecated_npm_package` | N/A (detected via `npm ls` output) | npm package with deprecation warning |
| `todo_without_issue` | `(?:\/\/\|#)\s*TODO(?:\([^)]*\))?:?\s+(?!.*(?:#\d+\|ISSUE-\|JIRA-\|TICKET-))` | TODO/FIXME without issue reference |
| `commented_out_code` | `^(?:\s*(?:\/\/\|#).*\n){10,}` | 10+ consecutive lines of commented code |
| `hack_comment` | `(?:\/\/\|#\|\/\*)\s*(?:HACK\|WORKAROUND\|XXX)\s*[:\-]?` | Explicit HACK/WORKAROUND/XXX marker |
| `outdated_callback` | `function\s+\w+\s*\([^)]*,\s*(?:callback\|cb\|done)\s*\)` | Callback pattern in async context |

---

## Output Format

**Process:** Header → Summary Table → Issues (by severity) → Cross-Cutting Insights → Test Recommendations. Display in terminal + write to file (generated or `--output-file`). Print: "Review saved to: [filepath]"

**Filename:** `yyyy-mm-dd_few-words-summary_review-type.md` — 2-4 lowercase-hyphenated words from file names or "staged-changes". Under 80 chars. Example: `2026-02-05_auth-middleware_code-review-deep.md`

**Header:** `## Code Review`, then `**Reviewed:** [N] file(s) | **Branch:** [branch-name]` and `**Review Depth:** [description]`. **No issues:** Header → "No issues found. All checks passed:" → categories → "Files reviewed:" → file list.

**Issues:** (1) Summary table: Categories x severity (Critical|Major|Minor|Suggestions) + Total row, reviewed categories only (9 deep, 4 quick). (2) Severity sections: `### Critical Issues (Must Fix)`, `### Major Issues (Should Fix)`, `### Minor Issues`, `### Suggestions` — only if issues exist. (3) Cross-Cutting Insights. (4) Test Recommendations. (5) Footer.

**Cross-Cutting Insights:** Omit if none. Format: `**[N]. [Title]** \`[Severity]\` \`[Primary Category] + [Secondary Category]\``, backtick `file:line`, description referencing related findings, then fix (diff/prompt).

**Issue Entry:** `**[N]. [Title]** \`[Severity]\` \`[Category]\`` + optional badge (`[2 agents]`/`[3+ agents]`), backtick `file:line`, description, `**Fix**:` with diff or prompt.

**Fix Formats:** One per unique issue. **Diff** (fix_type: diff): single-location, ≤10 lines, complete drop-in (no `...`), standard diff block with `-`/`+`. **Prompt** (fix_type: prompt): multi-location/structural. Label `**Fix prompt** (copy to Claude Code):` then blockquote with action verb, files, numbered changes.
