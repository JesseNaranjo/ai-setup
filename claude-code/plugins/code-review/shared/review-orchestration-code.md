# Code Review Orchestration

- Use git CLI (not GitHub CLI). Create todo list before starting.
- Cite issues with `file:line` (e.g., `src/utils.ts:42-48`). Quote exact AI instruction rules when violated.
- Paths relative to repo root. Line numbers reference file lines (not diff lines). Full file reviews (no changes): focus on clear bugs, not style.

## Agent Invocation

### Prompt Schema

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `MODE` | string | Yes | `thorough`, `gaps`, or `quick` |
| `project_type` | string | Yes | `nodejs`, `dotnet`, or both |
| `files_to_review` | list | Yes | See File Entry Schema below |
| `ai_instructions` | list | Conditional | Full content for architecture/compliance agents; summary-only for others |
| `related_tests` | list | Conditional | Only for bug-detection, technical-debt, test-coverage agents |
| `previous_findings` | list | Gaps only | Prior findings for dedup. Each: `title`, `file`, `line`, `range` (string or null), `category`, `severity` |
| `skill_instructions` | object | Optional | From `--skills`. Fields: `focus_areas`, `checklist` [{category, severity, items}], `auto_validate`, `false_positive_rules`, `methodology` {approach, steps, questions} |
| `additional_instructions` | string | Optional | Settings body + `--prompt` + orchestrator-injected rules |

### File Entry Schema

**Critical files** (has_changes: true, tier: "critical"): `path`, `diff` (unified), `full_content`

**Peripheral files** (staged — unchanged context, has_changes: false, tier: "peripheral"): `path`, `preview` (first ~50 lines), `line_count`, `full_content_available`: true (agent uses Read tool)

End prompt with: `Return findings as YAML per agent examples in your agent file.`

### Content Distribution

Per agent, append to `additional_instructions`:
1. Output schema, MODE definition, FP rules from Agent Common Instructions below
2. Category-specific FP rules — agent's category only
3. Language-specific checks — agent's anchor from detected language files
4. LSP diagnostic codes — agent's category rows + usage guidelines from lsp-integration.md

**Language anchors:** architecture→#architecture, bugs→#bugs, errors→#errors, performance→#performance, security→#security, debt→#debt, tests→#tests. Agents without anchors (api-contracts, compliance) skip step 3. Synthesis agents skip all distribution.

**LSP categories:** architecture→Architecture, bugs→Bugs, errors→Error Handling, performance→Performance, security→Security. If unavailable, skip.

**Selective:** `related_tests` → bug-detection, technical-debt, test-coverage only. Full `ai_instructions` → architecture, compliance only (others get path summary).

### Agent Common Instructions

**MODE:** thorough (all issues), gaps (subtle issues missed; dedup against previous_findings), quick (critical/merge-blocking only). Deep reviews skip pre-existing and silenced issues. Quick: only blocking issues, skip theoretical edge cases.

**False Positive Rules — do NOT flag:**
- Pre-existing issues not modified in current changes
- General quality unless required in AI instructions; test code unless reviewing tests; theoretical edge cases extremely unlikely in practice
- Code with lint-disable/suppress comments or documented suppressions

**Output Schema:** Each issue: `title`, `file`, `line`, `range` (string or null), `category`, `severity` (Critical/Major/Minor/Suggestion), `description`, `fix_type` (diff/prompt), `fix_diff` or `fix_prompt`. See agent file for category-specific extra fields.

### Category-Specific False Positive Rules

- **API Contracts**: Additive-only changes; beta/experimental APIs marked unstable; changes following established deprecation
- **Architecture**: Pragmatic compromises with clear justification; patterns overkill for project scale
- **Bug Detection**: Code with explicit comments explaining why it's correct
- **Compliance**: Explicit override comments; ambiguous rules with reasonable compliance; style preferences not stated as rules
- **Error Handling**: Errors intentionally ignored with explicit comments; logging-only catch blocks as intended
- **Performance**: Micro-optimizations without measurable impact; code that runs rarely
- **Security**: Internal-only code with no untrusted input; vulnerabilities mitigated elsewhere
- **Technical Debt**: Dependencies pinned for compatibility (documented); dead code conditionally compiled (build flags); TODO with issue tracking (TODO(#123)); workarounds with documented upstream bugs; class components in projects supporting older React
- **Test Coverage**: Code impractical to unit test (better for integration); code covered by higher-level tests; generated code/boilerplate; dead code to remove rather than test

## Gaps Mode

When MODE=gaps, agents receive `previous_findings` from thorough mode.

**Duplicate Detection:** Skip issues in same file within +/-5 lines of prior findings. Skip same issue type on same function/method. For range findings (lines A-B): skip zone = [A-5, B+5]. See Prompt Schema `previous_findings` field for schema.

**Constraints:** Only Major/Critical severity (skip Minor/Suggestion). Maximum 5 new findings per agent. Model: always Sonnet.

**Agents:** bug-detection, compliance, performance, security, technical-debt. See each agent file for category-specific gaps focus areas.

## Deep Code Review Sequence (19 invocations)

**Phase 1 — Thorough** (9 parallel): api-contracts, architecture, bug-detection, compliance, error-handling, performance, security, technical-debt, test-coverage. MODE: thorough. Models: Opus for architecture, bug-detection, performance, security, technical-debt; Sonnet for rest.
**CRITICAL: WAIT for ALL 9 before Phase 2.**

**Phase 2 — Gaps** (5 parallel Sonnet): bug-detection, compliance, performance, security, technical-debt. MODE: gaps. Input: Phase 1 findings as `previous_findings`. Content: diff only (agents Read for deeper analysis). Distribute Gaps Mode rules to each agent's `additional_instructions`.
**CRITICAL: WAIT for ALL 5 before Synthesis.**

**Synthesis** (5 parallel): 5 instances of synthesis-code-agent. Input: ALL Phase 1+2 findings. Content: file paths only (agents Read for cross-reference).
**CRITICAL: WAIT for Phase 1 AND Phase 2 fully complete before starting.**
Pairs: Architecture+Test Coverage ("Are architectural changes covered by tests?"), Bugs+Compliance ("Do compliance violations introduce or mask bugs?"), Bugs+Error Handling ("Do bugs have proper error handling in fix paths?"), Compliance+Technical Debt ("Do compliance violations indicate or worsen debt?"), Performance+Security ("Do security fixes introduce performance issues?")

**Post-review:** Validate all issues → filter invalid → deduplicate → generate report.

## Quick Code Review Sequence (7 invocations)

**Review** (4 parallel): bug-detection, error-handling, security, test-coverage. MODE: quick.

**Synthesis** (3 parallel): Bugs+Error Handling ("Do bugs have proper error handling?"), Bugs+Security ("Do security issues relate to bugs?"), Bugs+Test Coverage ("Are bugs covered by tests?")

**Post-review:** Same as deep.

## Model Selection (Authoritative)

Default: **Sonnet** for all agents and modes.
**Opus exceptions (thorough):** architecture, bug-detection, performance, security, technical-debt
**Opus exceptions (quick):** bug-detection, security
**Gaps:** Always Sonnet (no exceptions)

## Language-Specific Focus

Load ONLY for detected languages: `detected_languages.dotnet` → `${CLAUDE_PLUGIN_ROOT}/languages/dotnet.md`, `detected_languages.nodejs` → `${CLAUDE_PLUGIN_ROOT}/languages/nodejs.md`, `detected_frameworks.react` → also `${CLAUDE_PLUGIN_ROOT}/languages/react.md`

## Synthesis Invocation

Invoke synthesis agents multiple times in parallel with different category pairs. See `${CLAUDE_PLUGIN_ROOT}/agents/code/synthesis-code-agent.md`.

---

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

| Category | Model |
|----------|-------|
| API Contracts | Sonnet |
| Architecture | Opus |
| Bugs | Opus |
| Compliance | Sonnet |
| Error Handling | Sonnet |
| Performance | Opus |
| Security | Opus |
| Technical Debt | Sonnet |
| Test Coverage | Sonnet |

Cross-cutting insights always **Opus**.

## Aggregation

1. **Remove Invalid**: Filter INVALID issues
2. **Apply Downgrades**: Update severity for DOWNGRADE verdicts
3. **Deduplicate**: Merge same-file overlapping-range issues; keep most detailed description; use highest severity
4. **Consensus Detection**: Group by file, detect overlapping issues (same file, intersecting ranges: NOT (L2 < M1 OR M2 < L1)). Build clusters transitively (A overlaps B, B overlaps C → {A,B,C}). Merge each: highest severity, longest description, combined categories, badge (`[2 agents]` or `[3+ agents]`)

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
