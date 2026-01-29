# Content Gathering: Staged Commands

Shared content gathering logic for staged review commands (`/deep-code-review-staged`, `/quick-code-review-staged`).

## Process

Launch a Sonnet agent to gather the content to review:

### 1. Get Staged Diff

```bash
git diff --cached
```

### 2. Get Current Branch

```bash
git branch --show-current
```

### 3. Read Full File Content (Tiered)

For each staged file:
- **Changed files (has_changes=true)**: Read full file content (critical tier)
- **Unchanged files referenced in imports/context (has_changes=false)**: Read preview only (peripheral tier)

### 4. Read Related Test Files

Read related test files (from Context Discovery step) for context:
- **Node.js**: See `languages/nodejs.md` for test file patterns
- **.NET**: See `languages/dotnet.md` for test file patterns

Test files are always treated as critical tier.

### 5. Create Summary

Summarize what changes are being made:
- Number of files modified
- Brief description of the changes
- Detected project type(s)
- Related test files found
- Tier classification summary

## Output

Return to the review step:
- Current branch name
- For each file:
  - path
  - has_changes (true/false)
  - **tier**: "critical" or "peripheral"
  - For **critical** files (has_changes=true):
    - diff: full diff content
    - full_content: full file content
  - For **peripheral** files (has_changes=false):
    - preview: first 50 lines of file
    - line_count: total lines in file
    - full_content_available: true
- Related test files (always critical tier)
- Summary of what's being reviewed

### Tier Classification

| File Type | Tier | Rationale |
|-----------|------|-----------|
| Files with staged changes | critical | Primary review focus |
| Related test files | critical | Need full content for coverage analysis |
| AI instruction files | critical | Always fully loaded |
| Unchanged source files | peripheral | Context only - agents Read on-demand |

## Tiered Context Behavior

Staged reviews use tiered context to balance thoroughness with context efficiency:
- Changed files receive full content (critical tier)
- Unchanged files receive preview + metadata (peripheral tier)
- Agents can Read peripheral files on-demand if cross-file analysis discovers relevance
