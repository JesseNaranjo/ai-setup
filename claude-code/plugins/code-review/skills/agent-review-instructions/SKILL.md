---
name: agent-review-instructions
description: "Internal: MODE definitions, false positive rules, output schema for non-synthesis agents."
user-invocable: false
disable-model-invocation: true
---

**MODE:** thorough (all issues), gaps (subtle issues missed; dedup against previous_findings), quick (Critical/Major only; skip edge cases, theoretical issues, style). Deep reviews skip pre-existing and silenced issues.

**Gaps calibration:** Skip within ±5 lines of thorough findings or same issue type on same function/section. Major/Critical severity only. Maximum 5 new findings.

**False Positive Rules — do NOT flag:**
- Pre-existing issues not modified in current changes
- General quality unless required in AI instructions; test code unless reviewing tests; theoretical edge cases extremely unlikely in practice
- Code with lint-disable/suppress comments or documented suppressions

**Output Schema:** Each issue: `title`, `file`, `line`, `range` (string or null), `category`, `severity` (Critical/Major/Minor/Suggestion), `description`, `fix_type` (diff/prompt), `fix_diff` or `fix_prompt`. See agent file for category-specific extra fields.

Example:
```yaml
- title: "Missing null check on API response"
  file: "src/api/client.ts"
  line: 42
  range: "42-45"
  category: "Bugs"
  severity: "Major"
  description: "API response used without null check; throws TypeError when service returns 204"
  fix_type: "diff"
  fix_diff: |
    - const data = response.json();
    + const data = response.ok ? await response.json() : null;
```
