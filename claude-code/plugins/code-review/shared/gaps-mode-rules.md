# Gaps Mode Rules

When running in **gaps mode**, agents receive findings from the thorough mode review to avoid duplicates and focus on subtle issues.

## Input Format

Gaps mode agents receive `previous_findings` - a list of issues already flagged by thorough mode in the same category.

```yaml
previous_findings:
  - title: "SQL Injection in User Query"
    file: "src/data/UserRepository.cs"
    line: 45
    range: "45-48"
    severity: "Critical"
    category: "Security"
  - title: "Missing null check before access"
    file: "src/services/OrderService.cs"
    line: 112
    range: null
    severity: "Major"
    category: "Bugs"
```

**Fields explained:**
- `title`: Brief description of the issue
- `file`: Path to the file containing the issue
- `line`: Primary line number (always present)
- `range`: Line range as string (e.g., "45-48") or null if single line
- `severity`: Critical, Major, Minor, or Suggestion
- `category`: The review category (Security, Bugs, Performance, Compliance)

## Duplicate Detection Rules

**Skip issues matching prior findings.** An issue is a duplicate if:

1. **Same file AND within skip zone**

   **Skip Zone Calculation:**
   - Single-line finding (line N): skip zone = [N-5, N+5]
   - Range finding (lines A-B): skip zone = [A-5, B+5]

   **Overlap Detection:**
   - New single-line finding at L overlaps if: L >= zone_start AND L <= zone_end
   - New range finding X-Y overlaps if: NOT (Y < zone_start OR X > zone_end)

   **Examples:**
   - Prior at line 45 → zone [40, 50] → skip new findings touching lines 40-50
   - Prior at range "45-48" → zone [40, 53] → skip new findings touching lines 40-53
   - Prior at range "100-105" → zone [95, 110] → skip new findings touching lines 95-110

2. **Same issue type on same function/method**
   - Prior "null check" finding in `processOrder()` → skip other null check findings in same function

### What IS a Duplicate

| Prior Finding | New Finding | Duplicate? | Reason |
|---------------|-------------|------------|--------|
| SQL injection at `users.cs:45` | SQL injection at `users.cs:47` | **YES** | Same file, overlapping lines |
| Null check at `orders.cs:100` | Null check at `orders.cs:103` | **YES** | Same file, within ±5 lines |
| Missing auth at `api.cs:50-60` | Auth bypass at `api.cs:55` | **YES** | Line 55 is within range 50-60 |

### What is NOT a Duplicate

| Prior Finding | New Finding | Duplicate? | Reason |
|---------------|-------------|------------|--------|
| SQL injection at `users.cs:45` | SQL injection at `users.cs:120` | **NO** | Same file, but >5 lines apart |
| SQL injection at `users.cs:45` | XSS at `users.cs:47` | **NO** | Different vulnerability type |
| Null check at `orders.cs:100` | Null check at `payments.cs:100` | **NO** | Different files |
| N+1 query at `api.cs:50` | Memory leak at `api.cs:52` | **NO** | Different issue category |

## Prioritization Guidance

### What Gaps Mode SHOULD Look For

Focus on issues that thorough mode commonly misses:

**Security (gaps):**
- Second-order injection (stored XSS, delayed command execution)
- Authorization edge cases (role escalation, missing checks on related resources)
- Timing attacks and side channels
- Race conditions affecting security
- Error messages leaking sensitive information
- Weak randomness in security-critical code
- Insecure defaults

**Bugs (gaps):**
- Boundary conditions (empty arrays, zero values, max integer values)
- Race conditions in concurrent code
- State management issues (stale state, incorrect updates)
- Resource cleanup failures in error paths
- Time-of-check to time-of-use issues
- Integer overflow/underflow

**Performance (gaps):**
- Hidden N+1 queries (lazy loading, nested loops with DB calls)
- Memory retention through closures or event listeners
- Inefficient serialization/deserialization
- Cache invalidation issues
- Batch operation opportunities

**Compliance (gaps):**
- Rules with exceptions that weren't properly applied
- Inconsistent application of guidelines across files
- Context-dependent violations
- Subtle spirit-of-the-rule violations

### What Gaps Mode Should NOT Flag

- Issues already in `previous_findings` (duplicates)
- Pre-existing issues not introduced in the changes
- Theoretical issues requiring unrealistic conditions
- Issues in test code or development-only paths
- Micro-optimizations with no measurable impact

## Output Expectations

Gaps mode returns issues using the **same output schema** as thorough mode.

- Only return NEW issues not present in `previous_findings`
- Use identical YAML format from `${CLAUDE_PLUGIN_ROOT}/shared/output-schema-base.md`
- Apply same severity classification rules
- Include fix_type and fix_diff/fix_prompt as normal

**Example gaps mode output:**

```yaml
issues:
  - title: "Race condition in order status update"
    file: "src/services/OrderService.cs"
    line: 78
    range: "78-92"
    category: "Bugs"
    severity: "Major"
    description: "Concurrent requests can read stale order status between check and update"
    conditions: "Two requests for same order arrive within 10ms"
    impact: "Order processed twice, duplicate charges"
    fix_type: "prompt"
    fix_prompt: "Add optimistic locking to UpdateOrderStatus in src/services/OrderService.cs:78-92. Add a version/timestamp column check in the WHERE clause and handle concurrency conflicts."
```

This issue would only be returned if no bug was previously flagged in `OrderService.cs` lines 73-97 (±5 from 78-92).
