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

## Auto-Validation Patterns (Code)

Issues matching these patterns skip validation entirely and are marked `auto_validated: true`:

Apply unlabeled patterns to all languages. Apply [Node.js]/[React] patterns only for Node.js/React projects; apply [.NET] patterns only for .NET projects. Detected language from Context Discovery step.

### Architecture

- `circular_dependency` [Architecture/Major]: Circular import/dependency between modules
- `god_class_500_lines` [Architecture/Major]: Class/module exceeds 500 lines
- `function_10_params` [Architecture/Minor]: Function has 10+ parameters — `function\s+\w+\s*\([^)]*,\s*[^)]*,\s*[^)]*,\s*[^)]*,\s*[^)]*,\s*[^)]*,\s*[^)]*,\s*[^)]*,\s*[^)]*,\s*[^)]*\)`
- `direct_new_instantiation` [Architecture/Minor]: Direct instantiation of service/repository dependency — `(?:=\s*new\s+\w+Service\|=\s*new\s+\w+Repository)\s*\(`

### Bugs

- `blazor_improper_disposal` [Bugs/Major] [.NET]: Blazor component with IJSRuntime without IAsyncDisposable
- `empty_catch_block` [Bugs/Major]: Empty catch block (allows single comment) — `catch\s*\([^)]*\)\s*\{\s*(?:\/\/[^\n]*)?\s*\}`
- `missing_await` [Bugs/Major]: Async function call without await — `(?:const\|let\|var)\s+\w+\s*=\s*(?!await)[^;]*\basync\s+\w+\(`
- `null_dereference` [Bugs/Major]: Null access after optional chain or guard — `(?:\?\.\s*\w+\s*\(\)\s*\.)\|(?:\w+\s*&&\s*\w+\.\w+\s*\?\s*\w+\.\w+\.\w+)`
- `stale_useeffect_closure` [Bugs/Major] [React]: useEffect referencing state/props not in dependency array — `useEffect\s*\(\s*\(\)\s*=>\s*\{[^}]*\b(?:set\w+|dispatch)\b[^}]*\},\s*\[\s*\]\s*\)`

### Compliance

- `missing_authorize_attribute` [Compliance/Major] [.NET]: ASP.NET endpoint without [Authorize] attribute — `\[(?:Http(?:Get\|Post\|Put\|Delete\|Patch)\|Route)\][^[]*(?<!\[Authorize\])\s*public\s+(?:async\s+)?(?:Task<)?(?:IActionResult\|ActionResult)`
- `wrong_case_filename` [Compliance/Minor]: Filename violates project naming convention
- `explicit_must_violation` [Compliance/Major]: Code violates a MUST rule from AI instructions
- `missing_required_jsdoc` [Compliance/Minor] [Node.js]: Exported function without JSDoc comment — `export\s+(?:async\s+)?function\s+\w+\s*\([^)]*\)\s*(?::\s*\w+)?\s*\{(?!\s*/\*\*)`

### Error Handling

- `express_error_middleware_missing` [Error Handling/Major] [Node.js]: Express app without 4-argument error middleware — no `(err, req, res, next)` handler
- `unhandled_promise_rejection` [Error Handling/Major] [Node.js]: Promise chain without .catch() in request handler

### Performance

- `ef_missing_asnotracking` [Performance/Minor] [.NET]: EF Core read-only query without AsNoTracking()
- `missing_cancellation_token` [Performance/Minor] [.NET]: Async controller method without CancellationToken parameter
- `n_plus_one_query` [Performance/Major]: Database query inside iteration (N+1) — `(?:for\|while\|forEach\|\.map\|\.forEach)[\s\S]{0,200}(?:findOne\|findById\|query\|execute\|fetch)`
- `nested_loop_includes` [Performance/Major]: O(n²) nested loop with includes/indexOf — `for\s*\([^)]*\)[\s\S]*?for\s*\([^)]*\)[\s\S]*?\.(?:includes\|indexOf)\s*\(`
- `select_star_in_loop` [Performance/Major]: SELECT * inside a loop — `(?:for\|while\|forEach)[\s\S]*?SELECT\s+\*`

### Security

- `hardcoded_password` [Security/Critical]: Hardcoded password in assignment — `(?:password\|passwd\|pwd)\s*[:=]\s*['"][^'"]+['"]`
- `hardcoded_api_key` [Security/Critical]: Hardcoded API key — `(?:api[_-]?key\|apikey)\s*[:=]\s*['"][^'"]+['"]`
- `hardcoded_token` [Security/Critical]: Hardcoded auth token — `(?:token\|bearer\|auth[_-]?token\|access[_-]?token)\s*[:=]\s*['"][^'"]+['"]`
- `hardcoded_secret` [Security/Critical]: Hardcoded secret/private key — `(?:secret\|client[_-]?secret\|private[_-]?key)\s*[:=]\s*['"][^'"]+['"]`
- `hardcoded_credentials` [Security/Critical]: Hardcoded credentials object — `(?:credentials\|connection[_-]?string)\s*[:=]\s*['"][^'"]+['"]`
- `sql_injection_concat` [Security/Critical]: SQL with string concatenation of user input — `(?:SELECT\|INSERT\|UPDATE\|DELETE\|FROM\|WHERE).*[+]\s*(?:req\|request\|params\|query\|body\|input\|user)`
- `sql_injection_template` [Security/Critical]: SQL with template literal interpolation of user input — `(?:SELECT\|INSERT\|UPDATE\|DELETE\|FROM\|WHERE).*\$\{.*(?:req\|request\|params\|query\|body\|input\|user)`
- `eval_untrusted` [Security/Critical]: eval() with untrusted input — `eval\s*\(\s*(?:req\|request\|params\|query\|body\|input\|user)`
- `new_function_untrusted` [Security/Critical]: new Function() with untrusted input — `new\s+Function\s*\(\s*(?:req\|request\|params\|query\|body\|input\|user)`

### Technical Debt

- `deprecated_npm_package` [Technical Debt/Minor] [Node.js]: npm package with deprecation warning
- `todo_without_issue` [Technical Debt/Minor]: TODO/FIXME without issue reference — `(?:\/\/\|#)\s*TODO(?:\([^)]*\))?:?\s+(?!.*(?:#\d+\|ISSUE-\|JIRA-\|TICKET-))`
- `commented_out_code` [Technical Debt/Minor]: 10+ consecutive lines of commented code — `^(?:\s*(?:\/\/\|#).*\n){10,}`
- `hack_comment` [Technical Debt/Minor]: Explicit HACK/WORKAROUND/XXX marker — `(?:\/\/\|#\|\/\*)\s*(?:HACK\|WORKAROUND\|XXX)\s*[:\-]?`
- `outdated_callback` [Technical Debt/Minor] [Node.js]: Callback pattern in async context — `function\s+\w+\s*\([^)]*,\s*(?:callback\|cb\|done)\s*\)`

---

## Output Format

**Process:** Header → Summary Table → Issues (by severity) → Cross-Cutting Insights → Test Recommendations. Display in terminal + write to file (generated or `--output-file`). Print: "Review saved to: [filepath]"

**Filename:** `yyyy-mm-dd_few-words-summary_review-type.md` — 2-4 lowercase-hyphenated words from file names or "staged-changes". Under 80 chars. Example: `2026-02-05_auth-middleware_code-review-deep.md`

**Header:** `## Code Review`, then `**Reviewed:** [N] file(s) | **Branch:** [branch-name]` and `**Review Depth:** [description]`. **No issues:** Header → "No issues found. All checks passed:" → categories → "Files reviewed:" → file list.

**Issues:** (1) Summary table: Categories x severity (Critical|Major|Minor|Suggestions) + Total row, reviewed categories only (9 deep, 4 quick). (2) Severity sections: `### Critical Issues (Must Fix)`, `### Major Issues (Should Fix)`, `### Minor Issues`, `### Suggestions` — only if issues exist. (3) Cross-Cutting Insights. (4) Test Recommendations. (5) Footer.

**Issue Entry:** `**[N]. [Title]** \`[Severity]\` \`[Category]\`` + optional badge (`[2 agents]`/`[3+ agents]`), backtick `file:line`, description, `**Fix**:` with diff or prompt. **Cross-cutting insights:** same format but `\`[Primary Category] + [Secondary Category]\`` instead of single category; omit section if none.

**Fix Formats:** One per unique issue. **Diff** (fix_type: diff): single-location, ≤10 lines, complete drop-in (no `...`), standard diff block with `-`/`+`. **Prompt** (fix_type: prompt): multi-location/structural. Label `**Fix prompt** (copy to Claude Code):` then blockquote with action verb, files, numbered changes.
