# Base Output Schema

All review agents return issues using this base YAML schema. Individual agents extend this with category-specific fields.

Reference `shared/severity-definitions.md` for severity classification and `shared/output-format.md` for final output formatting.

## Required Base Fields

```yaml
issues:
  - title: "Brief descriptive title"
    file: "path/to/file.ts"
    line: 42                    # Primary line number (always required)
    range: "42-50"              # Optional: line range for multi-line issues
    category: "Security"        # Category name (see below)
    severity: "Critical"        # Critical | Major | Minor | Suggestion
    description: "What the issue is and why it matters"
    fix_type: "diff"            # diff | prompt (required)
    fix_diff: |                 # Include ONLY if fix_type is "diff"
      - old code line
      + new code line
    fix_prompt: "..."           # Include ONLY if fix_type is "prompt"
```

### Field Details

**title** (required): Brief description in 5-10 words. Should clearly identify the issue type.
- Good: "SQL injection via string concatenation"
- Bad: "Security issue" or "Problem in code"

**file** (required): Relative path from repository root.

**line** (required): Primary line number where the issue occurs. Use the starting line if the issue spans multiple lines.

**range** (optional): Line range as string for multi-line issues. Format: `"start-end"`.
- Use `line` alone for single-line issues: `line: 42`
- Use both `line` and `range` for multi-line: `line: 42, range: "42-50"`

**category** (required): One of:
- `"Security"` - Security vulnerabilities
- `"Bugs"` - Logical errors, runtime issues
- `"Performance"` - Performance issues
- `"Compliance"` - AI Agent Instructions violations
- `"Architecture"` - Design and structure issues
- `"API Contracts"` - API compatibility issues
- `"Error Handling"` - Error handling gaps
- `"Test Coverage"` - Missing tests

**severity** (required): See `shared/severity-definitions.md` for full criteria.
- `"Critical"` - Must fix before merge
- `"Major"` - Should fix before merge
- `"Minor"` - Can merge, fix soon
- `"Suggestion"` - Optional improvement

**description** (required): Explanation of the issue. Include:
- What the problem is
- Why it matters (impact, risk)
- When it manifests (conditions, scenarios)

## Fix Type Selection

Every issue MUST include either `fix_diff` OR `fix_prompt`, determined by `fix_type`.

### Use `diff` When:

- **Single location** - Fix is in one place in one file
- **≤10 lines changed** - Small, focused change
- **Exact replacement known** - No decisions needed
- **Drop-in replacement** - No dependencies on other changes

**Example: SQL injection fix**
```yaml
fix_type: "diff"
fix_diff: |
  - const result = db.query(`SELECT * FROM users WHERE id = ${userId}`);
  + const result = db.query('SELECT * FROM users WHERE id = ?', [userId]);
```

**Example: Null check addition**
```yaml
fix_type: "diff"
fix_diff: |
  + if (user == null) {
  +   throw new Error('User not found');
  + }
    return user.email;
```

### Use `prompt` When:

- **Multi-location changes** - Fix spans multiple files or locations
- **Structural refactoring** - Extracting classes, reorganizing code
- **Requires decisions** - Naming choices, placement decisions
- **>10 lines would change** - Large-scale modifications
- **Context-dependent** - Fix depends on project patterns/preferences

**Example: CSRF protection across endpoints**
```yaml
fix_type: "prompt"
fix_prompt: "Add CSRF protection to all state-changing endpoints in src/api/account.ts. Install csurf middleware, generate tokens on GET requests, validate tokens on POST/PUT/DELETE. Update frontend forms to include the CSRF token."
```

**Example: Service extraction**
```yaml
fix_type: "prompt"
fix_prompt: "Extract authentication logic from UserService into a new AuthService class. Create src/services/AuthService.ts, move login(), logout(), and validateToken() methods, update all imports in consuming files."
```

## Category-Specific Extensions

Each agent category may add extra fields beyond the base schema:

**Security** (see `agents/security-agent.md`):
```yaml
attack_vector: "How an attacker could exploit this"
impact: "What damage could result"
```

**Bugs** (see `agents/bug-detection-agent.md`):
```yaml
conditions: "When this bug occurs"
impact: "What happens when the bug triggers"
```

**Performance** (see `agents/performance-agent.md`):
```yaml
complexity: "Time/space complexity (e.g., O(n^2))"
scale: "At what data size this becomes a problem"
impact: "Expected performance degradation"
```

**Compliance** (see `agents/compliance-agent.md`):
```yaml
rule_violated: "Exact quote from instruction file"
rule_source: "Path to instruction file with line number"
```

## Complete Examples

### Security Issue with Diff Fix

```yaml
issues:
  - title: "SQL injection via string concatenation"
    file: "src/data/UserRepository.cs"
    line: 45
    range: "45-48"
    category: "Security"
    severity: "Critical"
    description: "User input directly concatenated into SQL query. Attacker can extract or modify any database data."
    attack_vector: "Submit malicious SQL in userId parameter: ' OR 1=1 --"
    impact: "Full database access, data exfiltration, data modification"
    fix_type: "diff"
    fix_diff: |
      - var query = "SELECT * FROM Users WHERE Id = " + userId;
      - var result = cmd.ExecuteQuery(query);
      + var query = "SELECT * FROM Users WHERE Id = @id";
      + cmd.Parameters.AddWithValue("@id", userId);
      + var result = cmd.ExecuteQuery(query);
```

### Bug Issue with Prompt Fix

```yaml
issues:
  - title: "Race condition in concurrent order processing"
    file: "src/services/OrderService.cs"
    line: 45
    range: "45-78"
    category: "Bugs"
    severity: "Critical"
    description: "Multiple concurrent requests can process the same order twice due to read-modify-write race."
    conditions: "Two requests for same orderId arrive within 10ms window"
    impact: "Duplicate charges, inventory inconsistency, order fulfillment errors"
    fix_type: "prompt"
    fix_prompt: "Add distributed locking to processOrder in src/services/OrderService.cs:45-78. Use Redis-based lock with orderId as key, 30-second TTL, and retry logic. Wrap the entire order processing in a try-finally to ensure lock release."
```

## Edge Cases

**When to use `line` vs `range`:**
- Single statement issue: `line: 42` only
- Multi-line block (function, loop): `line: 42, range: "42-55"`
- If in doubt, include the range for clarity

**When description needs extra context:**
- Reference related code if the issue involves interactions between components
- Mention framework-specific behavior if relevant (e.g., React hooks rules)
- Include error messages if the issue causes specific failures
