# Severity Definitions

This file defines the canonical severity levels used across all code review agents.
Reference this file to ensure consistent severity classification.

---

## Critical

Issues that must be fixed before merge. These represent:

- **Exploitable security vulnerabilities**: SQL injection, XSS, authentication bypass, command injection
- **Data loss or corruption risks**: Unhandled writes, race conditions affecting data integrity
- **System crashes or service unavailability**: Null pointer dereferences in critical paths, infinite loops
- **Broken core functionality**: Logic errors that prevent primary features from working

**Action**: Block merge until resolved.

---

## Major

Issues that should be fixed but may not block merge in time-sensitive situations:

- **Security issues requiring specific conditions**: CSRF without sensitive actions, information disclosure requiring authentication
- **Significant performance degradation**: O(n²) in hot paths, memory leaks over time, N+1 queries
- **Logic errors affecting key workflows**: Incorrect calculations, wrong branch conditions
- **Breaking API changes**: Removed endpoints, changed response shapes, incompatible parameter changes

**Action**: Fix before merge unless time-critical with documented exception.

---

## Minor

Issues that improve code quality but don't affect correctness:

- **Code quality issues**: Duplicated code, overly complex functions, poor naming
- **Non-critical performance inefficiencies**: Suboptimal but not problematic performance
- **Style/pattern inconsistencies**: Deviations from project conventions
- **Missing edge case handling**: Uncommon scenarios without explicit handling

**Action**: Fix when convenient or in follow-up PR.

---

## Suggestion

Opportunities for improvement, not problems:

- **Improvement opportunities**: Better algorithms, cleaner patterns
- **Better alternatives exist**: More idiomatic approaches available
- **Documentation gaps**: Missing comments on complex logic
- **Test coverage recommendations**: Additional test cases that would improve confidence

**Action**: Consider for future improvement.

---

## Severity by Category Examples

| Category | Critical | Major | Minor | Suggestion |
|----------|----------|-------|-------|------------|
| **Security** | SQL injection | CSRF on sensitive action | Missing CSP header | Add rate limiting |
| **Bug** | Null deref in payment flow | Wrong calculation | Off-by-one in pagination | Simplify conditional |
| **Performance** | Infinite loop | N+1 query | Unnecessary re-render | Use memo() |
| **Architecture** | Circular dependency breaking build | Tight coupling | Missing abstraction | Consider pattern X |
| **API** | Removed required endpoint | Changed response type | Inconsistent naming | Add OpenAPI docs |
| **Error Handling** | Crash on invalid input | Missing retry on transient error | Generic error message | Add error codes |
| **Test Coverage** | No tests for auth flow | Missing edge case test | Low branch coverage | Add property tests |
| **Compliance** | Violates MUST rule | Violates SHOULD rule | Inconsistent with guideline | Could follow suggestion |
| **Technical Debt** | Deprecated dependency with CVE | Major version 2+ behind | TODO without tracking | Modernization opportunity |

---

## Technical Debt Severity Guidelines

- **Critical**: Deprecated dependency with known vulnerabilities (CVE), removed API usage requiring immediate migration, blocking modernization path
- **Major**: Major version 2+ behind with breaking changes pending, scalability blocker in production, extensive workaround code affecting multiple files
- **Minor**: Outdated patterns that still work correctly, TODO/FIXME without urgency or tracking, minor documentation gaps
- **Suggestion**: Code modernization opportunity, style improvements, optional refactoring for consistency
