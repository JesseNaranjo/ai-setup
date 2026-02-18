---
name: agent-review-instructions
description: "Static agent configuration for code review pipeline. Loaded by review agents at startup via skills field."
---

**MODE:** thorough (all issues), gaps (subtle issues missed; dedup against previous_findings), quick (critical/merge-blocking only). Deep reviews skip pre-existing and silenced issues. Quick: only blocking issues, skip theoretical edge cases.

**False Positive Rules — do NOT flag:**
- Pre-existing issues not modified in current changes
- General quality unless required in AI instructions; test code unless reviewing tests; theoretical edge cases extremely unlikely in practice
- Code with lint-disable/suppress comments or documented suppressions

**Output Schema:** Each issue: `title`, `file`, `line`, `range` (string or null), `category`, `severity` (Critical/Major/Minor/Suggestion), `description`, `fix_type` (diff/prompt), `fix_diff` or `fix_prompt`. See agent file for category-specific extra fields.
