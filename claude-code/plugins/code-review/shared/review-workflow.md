# Shared 10-Agent Review Workflow

This document defines the comprehensive code review workflow used by both `/code-review-files` and `/code-review-staged` commands.

## Severity Classification

All issues MUST be classified using these severity levels:

| Severity | Definition | Action Required |
|----------|------------|-----------------|
| **Critical** | Data loss, security breach, production outage risk | Must fix before merge |
| **Major** | Significant bug or violation affecting functionality | Should fix before merge |
| **Minor** | Small issue that should be addressed | Can merge, fix soon |
| **Suggestion** | Recommended improvement for future consideration | Optional |

## Language-Specific Focus Areas

### Node.js (JavaScript/TypeScript)

Detect Node.js projects by checking for `package.json` in the repository.

| Category | Language-Specific Focus |
|----------|------------------------|
| **Bugs** | Unhandled promise rejections, `undefined`/`null` issues, incorrect `this` binding, async/await pitfalls, type coercion bugs |
| **Security** | Prototype pollution, ReDoS, dynamic code execution, insecure dependencies, JWT validation, XSS via template literals |
| **Performance** | Event loop blocking, memory leaks from closures/event listeners, inefficient array methods (forEach vs for), missing stream usage for large data |
| **Architecture** | CommonJS vs ESM issues, circular imports, React hooks rules violations, improper TypeScript typing |
| **Error Handling** | Unhandled promise rejections, missing `.catch()`, missing error boundaries (React), swallowed errors |
| **Test Files** | `*.test.ts`, `*.spec.ts`, `*.test.js`, `*.spec.js`, `*-test.js`, `*-spec.js`, `__tests__/` and `tests/` directories |

### .NET (C#)

Detect .NET projects by checking for `*.csproj`, `*.sln`, or `*.slnx` files in the repository.

| Category | Language-Specific Focus |
|----------|------------------------|
| **Bugs** | Null reference exceptions, `IDisposable` not disposed, async deadlocks (`Result`/`Wait()` on async), LINQ deferred execution issues |
| **Security** | SQL injection via string concatenation, insecure deserialization, hardcoded connection strings, missing `[Authorize]` attributes, path traversal |
| **Performance** | Boxing/unboxing overhead, LINQ in hot loops, excessive allocations, missing `ConfigureAwait(false)` in libraries, N+1 EF queries |
| **Architecture** | DI anti-patterns (service locator), missing interfaces for testability, controller bloat, improper layering violations |
| **Error Handling** | Missing exception handling, swallowed exceptions (empty catch), improper `Task` handling, missing try-finally for cleanup |
| **Test Files** | `*.Tests.cs`, `*Tests/` projects, `*.UnitTests/` projects, `tests/` directories |

---

## Review Agents (Step 4)

Launch ALL 10 agents in parallel. Each agent returns a list of issues with:
- Issue title
- File path and line range
- Description
- Category (from agent's focus area)
- Suggested severity level

### Agent Definitions

**Agents 1-2: Compliance with CLAUDE.md and other AI Agent Instructions (Opus)**

```
Agent 1: AI Agent Instructions Compliance Checker A
Model: Opus
Focus: Standards adherence

Review the code for compliance with CLAUDE.md and other AI Agent Instructions. For each file being reviewed, only consider CLAUDE.md and other AI Agent Instructions files that share a file path with the file or its parent directories.

For each violation:
- Quote the exact CLAUDE.md and other AI Agent Instructions rule being violated
- Cite the file path and line numbers
- Explain why this is a violation
- Classify severity (usually Major for explicit rule violations)

Only flag clear, unambiguous violations where you can quote the exact rule.
```

```
Agent 2: AI Agent Instructions Compliance Checker B
Model: Opus
Focus: Independent verification of standards adherence

Independently review for compliance with CLAUDE.md and other AI Agent Instructions using the same approach as Agent 1. Do not coordinate with Agent 1 - this provides redundancy and catches issues one agent might miss.
```

**Agents 3-4: Bug Detection (Opus)**

```
Agent 3: Bug Detection - Logical Errors
Model: Opus
Focus: Runtime bugs, null references, off-by-one errors

Scan for logical bugs that will cause incorrect behavior at runtime:
- Null/undefined reference errors
- Off-by-one errors in loops/arrays
- Incorrect conditionals (wrong operator, inverted logic)
- Type mismatches causing runtime failures
- Uninitialized variables
- Incorrect function return values

Node.js specific: Unhandled promises, async/await misuse, `this` binding
.NET specific: Null reference exceptions, IDisposable leaks, async deadlocks

Only flag bugs you are confident will cause incorrect behavior. Ignore theoretical edge cases.
```

```
Agent 4: Bug Detection - Edge Cases & State
Model: Opus
Focus: Boundary conditions, race conditions, state management

Scan for edge case and state-related bugs:
- Boundary condition failures (empty arrays, zero values, max values)
- Race conditions in concurrent code
- State management issues (stale state, incorrect updates)
- Resource cleanup failures
- Incorrect error recovery paths

Node.js specific: Event loop blocking, memory leaks from closures
.NET specific: LINQ deferred execution issues, improper Task handling

Focus on high-confidence issues in the changed code.
```

**Agent 5: Security (Opus)**

```
Agent 5: Security Analysis
Model: Opus
Focus: Injection, authentication, secrets, OWASP issues

Analyze for security vulnerabilities:
- Injection attacks (SQL, command, XSS, template)
- Authentication/authorization bypasses
- Hardcoded secrets, API keys, passwords
- Insecure cryptographic practices
- Path traversal vulnerabilities
- Insecure deserialization
- Missing input validation on security boundaries

Node.js specific: Prototype pollution, ReDoS, dynamic code execution, JWT issues
.NET specific: SQL via string concat, missing [Authorize], connection strings

Classify severity:
- Critical: Direct exploitation risk, data breach potential
- Major: Requires specific conditions but exploitable
- Minor: Defense-in-depth issues
```

**Agent 6: Performance (Opus)**

```
Agent 6: Performance Analysis
Model: Opus
Focus: Complexity, memory, hot paths, N+1 queries

Analyze for performance issues:
- O(n^2) or worse algorithms where O(n) is possible
- Memory leaks or excessive allocations
- N+1 query patterns in database access
- Blocking operations in async contexts
- Inefficient data structure usage
- Unnecessary object creation in hot paths

Node.js specific: Event loop blocking, inefficient array methods, missing streams
.NET specific: Boxing/unboxing, LINQ in loops, missing ConfigureAwait

Only flag issues that will have measurable impact. Ignore micro-optimizations.
Severity: Critical only if causes outages, usually Major or Minor.
```

**Agent 7: Architecture (Sonnet)**

```
Agent 7: Architecture Review
Model: Sonnet
Focus: Coupling, patterns, SOLID principles

Review for architectural issues:
- High coupling between unrelated components
- SOLID principle violations
- Anti-patterns (god objects, feature envy, shotgun surgery)
- Layer violations (presentation accessing data directly)
- Missing abstractions that hurt maintainability

Node.js specific: Circular imports, CommonJS/ESM mixing, React hooks violations
.NET specific: DI anti-patterns, missing interfaces, controller bloat

Severity: Usually Minor or Suggestion unless causing immediate problems.
```

**Agent 8: API & Contracts (Sonnet)**

```
Agent 8: API & Contract Analysis
Model: Sonnet
Focus: Breaking changes, compatibility, interface contracts

Analyze for API and contract issues:
- Breaking changes to public APIs (removed methods, changed signatures)
- Backward compatibility issues
- Interface contract violations
- Inconsistent API patterns
- Missing versioning on breaking changes
- Schema/type changes affecting consumers

Severity:
- Critical: Breaking change without migration path
- Major: Breaking change with workaround
- Minor: Inconsistency, documentation gaps
```

**Agent 9: Error Handling (Sonnet)**

```
Agent 9: Error Handling Review
Model: Sonnet
Focus: Try/catch gaps, resilience, error recovery

Review error handling practices:
- Missing error handling for operations that can fail
- Swallowed exceptions (empty catch blocks)
- Error messages that leak sensitive information
- Missing cleanup/finally blocks
- Incorrect error propagation
- Missing retry logic for transient failures

Node.js specific: Unhandled rejections, missing .catch(), error boundaries
.NET specific: Missing exception handling, improper Task handling

Severity: Major if can cause crashes/data loss, otherwise Minor.
```

**Agent 10: Test Coverage (Sonnet)**

```
Agent 10: Test Coverage Analysis
Model: Sonnet
Focus: Missing tests, test quality, test suggestions

Analyze test coverage for the changed code:
- New code paths without corresponding tests
- Edge cases not covered by existing tests
- Modified logic that invalidates existing tests
- Missing integration tests for API changes
- Test quality issues (no assertions, testing implementation details)

Detect test files based on project type:
- Node.js: *.test.ts, *.spec.ts, *.test.js, *.spec.js, __tests__/, tests/
- .NET: *.Tests.cs, *Tests/ projects, *.UnitTests/, tests/

Output: List of specific test cases that should be added, with:
- What to test
- Expected behavior
- Suggested test file location

Severity: Usually Suggestion, Minor if critical path lacks tests.
```

---

## Validation (Step 5)

For EACH issue found by ALL agents (not just bugs), launch validation subagents:

**Validation Rules:**
- Use Opus validators for: Bugs (3-4), Security (5), Performance (6)
- Use Sonnet validators for: AI Agent Instructions (1-2), Architecture (7), API (8), Errors (9), Tests (10)

**Validator Prompt Template:**
```
Validate this issue by checking if it is truly present in the code:

Issue: [title]
File: [path:lines]
Category: [category]
Description: [description]

Your task:
1. Read the relevant file(s) to verify the issue exists
2. Check if this is actually a problem or a false positive
3. Verify severity classification is appropriate
4. Return: VALID, INVALID, or DOWNGRADE (with new severity)

Common false positives to check:
- Pre-existing issues not introduced in these changes
- Code that appears wrong but has valid context
- Issues already handled elsewhere
- Lint-ignore comments silencing intentional violations
```

---

## Aggregation (Step 6)

After validation, perform deduplication and consensus scoring:

1. **Remove invalid issues**: Filter out issues marked INVALID by validators
2. **Apply severity downgrades**: Update severity for issues marked DOWNGRADE
3. **Deduplicate**: Merge issues flagged by multiple agents for same file+line range
4. **Consensus scoring**: Issues flagged by 2+ agents get higher confidence badge

---

## Output Format (Step 7)

Generate the review output in this format. Write to BOTH terminal AND file.

### If NO issues were found:

```markdown
## Code Review

**Reviewed:** [N] file(s) | **Branch:** [branch-name]
**Review Depth:** Comprehensive (10-agent analysis)

No issues found. All checks passed:
- Compliance with CLAUDE.md and other AI Agent Instructions
- Bug detection (logical errors, edge cases)
- Security analysis
- Performance review
- Architecture patterns
- API compatibility
- Error handling
- Test coverage

Files reviewed:
[list files]
```

### If issues WERE found:

```markdown
## Code Review

**Reviewed:** [N] file(s) | **Branch:** [branch-name]
**Review Depth:** Comprehensive (10-agent analysis)

### Summary

| Category | Critical | Major | Minor | Suggestions |
|----------|----------|-------|-------|-------------|
| AI Agent Instructions | 0 | 0 | 0 | 0 |
| Bugs | 0 | 0 | 0 | 0 |
| Security | 0 | 0 | 0 | 0 |
| Performance | 0 | 0 | 0 | 0 |
| Architecture | 0 | 0 | 0 | 0 |
| API/Contracts | 0 | 0 | 0 | 0 |
| Error Handling | 0 | 0 | 0 | 0 |
| Test Coverage | 0 | 0 | 0 | 0 |
| **Total** | **0** | **0** | **0** | **0** |

### Critical Issues (Must Fix)

[List critical issues - empty section if none]

### Major Issues (Should Fix)

[List major issues - empty section if none]

### Minor Issues

[List minor issues - empty section if none]

### Suggestions

[List suggestions - empty section if none]

### Test Recommendations

[Aggregated test case suggestions from Agent 10]

---
Review saved to: [filepath]
```

### Issue Entry Format:

```markdown
**[N]. [Issue title]** `[Severity]` `[Category]` [Consensus badge if 2+ agents]
`path/to/file.ts:line-range`

[Description of the issue]

[For small fixes (up to 5 lines), include suggestion block:]
```suggestion
[corrected code]
```

[For larger fixes, include prompt:]
```
Fix [file:line]: [brief description of issue and suggested fix]
```
```

---

## False Positive Guidelines

Do NOT flag these (across all agents):

- Pre-existing issues not introduced in the changes being reviewed
- Code that appears problematic but is actually correct in context
- Pedantic nitpicks that a senior engineer would not flag
- Issues that a linter/type checker will catch
- General quality concerns not explicitly required by CLAUDE.md and other AI Agent Instructions
- Issues with explicit ignore comments (lint-disable, etc.)
- Theoretical issues that require specific conditions to manifest
- Subjective style preferences not mandated by guidelines

---

## Notes

- Use git CLI for repository interactions (not GitHub CLI)
- Always cite file paths and line numbers for each issue
- Quote exact CLAUDE.md and other AI Agent Instructions rules when citing violations
- File paths should be relative to repository root
- Line numbers reference lines in working copy (not diff line numbers)
- Maintain a todo list to track review progress
- Each issue should appear only once in the output (deduplicate)
