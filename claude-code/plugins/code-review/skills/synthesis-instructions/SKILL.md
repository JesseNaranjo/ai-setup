---
name: synthesis-instructions
description: "Internal: input format, review process, output schema for synthesis agents."
user-invocable: false
disable-model-invocation: true
---

## Input

Receives `synthesis_input` with:
- `category_a.findings` - Findings from first category
- `category_b.findings` - Findings from second category
- `cross_cutting_question` - The question to answer
- `files_content` - File diffs and full content for context

## Review Process

Map findings to files. For each file with findings from both categories: analyze interactions per agent's Step 2 patterns. Check if proposed fixes introduce issues in the other category. Report only cross-cutting insights not caught by individual agents.

## Output Schema

Return `cross_cutting_insights` as a YAML list. Use category keys from the agent's Category Key Mapping. Required fields per insight: `title`, `related_findings` (both category keys required — reject single-category insights), `insight`, `category` (Title Case), `severity` (Critical|Major|Minor|Suggestion), `file`, `line`, `fix_type` (diff|prompt), `fix_diff` or `fix_prompt`.

Example:
```yaml
- title: "Auth middleware removed from hot path without security test"
  related_findings:
    security: "Missing authentication on /api/users endpoint"
    test_coverage: "No integration test for /api/users auth flow"
  insight: "Security fix adds auth middleware but no test covers the auth requirement; removal would silently reintroduce vulnerability"
  category: "Security + Test Coverage"
  severity: "Major"
  file: "src/routes/users.ts"
  line: 15
  fix_type: "prompt"
  fix_prompt: "Add integration test verifying /api/users returns 401 without valid auth token"
```

## Guidelines

**DO flag**: Issues spanning two categories, ripple effects from proposed fixes, gaps where category A's finding implies category B should have found something.

**DO NOT flag**: Issues already caught by either input category, theoretical interactions with no practical impact, duplicates, issues requiring information outside reviewed content, insights where only one category has a related finding.
