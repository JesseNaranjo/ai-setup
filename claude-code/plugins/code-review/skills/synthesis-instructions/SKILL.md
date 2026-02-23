---
name: synthesis-instructions
description: "Input format, review process, output schema, and guidelines for cross-cutting synthesis agents."
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

### Step 1: Map Findings to Files

Map findings to files. For each file with findings from both categories: analyze interactions per agent's Step 2 patterns. Check if proposed fixes introduce issues in the other category. Report only cross-cutting insights not caught by individual agents.

### Step 2: Analyze Cross-Category Interactions

For each file with findings from both categories, analyze how findings interact. See agent file for domain-specific interaction patterns.

## Output Schema

Return cross-cutting insights as a YAML list. Use category keys from the agent's Category Key Mapping.

```yaml
cross_cutting_insights:
  - title: "Brief descriptive title"
    related_findings:
      <category_a_key>: "Title of related finding from Category A"
      <category_b_key>: "Title of related finding from Category B"
    # Use lowercase category keys from agent's Category Key Mapping
    # Both related findings are REQUIRED. If only one category has a finding, don't flag.
    insight: "What the cross-cutting concern is and why it matters"
    category: "<Primary Category>"  # Title Case from agent's Category Key Mapping
    severity: "Critical|Major|Minor|Suggestion"
    file: "path/to/file"
    line: 42
    fix_type: "diff|prompt"
    fix_diff: |  # if fix_type is diff
      - old line
      + new line
    fix_prompt: "..."  # if fix_type is prompt
```

## Guidelines

**DO flag**: Issues spanning two categories, ripple effects from proposed fixes, gaps where category A's finding implies category B should have found something.

**DO NOT flag**: Issues already caught by either input category, theoretical interactions with no practical impact, duplicates, issues requiring information outside reviewed content, insights where only one category has a related finding.
