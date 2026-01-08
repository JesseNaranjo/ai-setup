# Code Review Files Plugin

Automated code review for specific files using multiple specialized agents with confidence-based scoring to filter false positives. Reviews uncommitted changes if present, or the entire file if no changes exist.

## Overview

The Code Review Files Plugin provides targeted code review for specific files in your git repository. Unlike the code-review-changes plugin which reviews all uncommitted changes, this plugin lets you specify exactly which files to review. It intelligently handles two scenarios:

1. **Files with uncommitted changes**: Reviews only the changes (like code-review-changes)
2. **Files without changes**: Reviews the entire file content

This makes it ideal for reviewing specific files regardless of their change status.

## Commands

### `/code-review-files`

Performs automated code review on specified files using multiple specialized agents.

**What it does:**
1. Validates input files and checks for uncommitted changes
2. Categorizes files into "with changes" and "without changes"
3. Gathers relevant CLAUDE.md guideline files from the repository
4. Collects diff content for changed files, full content for unchanged files
5. Launches 4 parallel agents to independently review:
   - **Agents #1 & #2**: Audit for CLAUDE.md compliance
   - **Agent #3**: Scan for obvious bugs
   - **Agent #4**: Analyze for security issues and incorrect logic
6. Validates each issue with follow-up agents
7. Filters out low-confidence issues
8. Outputs review to terminal AND saves to markdown file

**Usage:**
```bash
/code-review-files <file1> [file2] [file3] ... [--output-file <path>]
```

**Arguments:**
- `<file>`: One or more file paths to review (required)
- `--output-file <path>`: Save the review to a custom file path (default: `.code-review-files.md` in repo root)

**Examples:**
```bash
# Review a single file
/code-review-files src/utils.ts

# Review multiple files
/code-review-files src/api/handler.ts src/models/user.ts src/services/auth.ts

# Review with custom output location
/code-review-files --output-file reviews/feature-review.md src/feature.ts

# Review a mix of changed and unchanged files
/code-review-files src/new-feature.ts src/existing-utils.ts
```

**Features:**
- Targeted file review (specify exactly what to review)
- Smart detection of changed vs unchanged files
- Reviews changes when present, full file when not
- Multiple independent agents for comprehensive review
- Confidence-based validation reduces false positives
- CLAUDE.md compliance checking with explicit guideline verification
- Bug detection focused on the content being reviewed
- Outputs to both terminal and file for reference
- Local file path references with line numbers

**Review output format:**
```markdown
## File Code Review

Reviewed 3 file(s):
- With changes: src/feature.ts, src/api.ts
- Full file review: src/utils.ts

### Issues Found: 2

**1. Missing error handling** (Bug)
`src/feature.ts:42-45`

The async function doesn't handle promise rejection...

```suggestion
try {
  await fetchData();
} catch (error) {
  console.error('Failed to fetch:', error);
}
```

**2. CLAUDE.md violation** (CLAUDE.md violation)
`src/utils.ts:78`

Violates rule: "Always validate input parameters"
...

---
Review saved to: .code-review-files.md
```

**False positives filtered:**
- Pre-existing issues in changed files (not introduced in changes)
- Code that looks like a bug but isn't
- Pedantic nitpicks
- Issues linters will catch
- General quality issues (unless in CLAUDE.md)
- Issues with lint ignore comments

## Comparison with code-review-changes

| Feature | code-review-changes | code-review-files |
|---------|-------------------|-------------------|
| Scope | All uncommitted changes | Specific files only |
| File selection | Automatic (all changed) | Manual (user specifies) |
| Unchanged files | Not reviewed | Reviewed in full |
| Use case | Pre-commit review | Targeted file review |

**When to use code-review-files:**
- Review specific files you're concerned about
- Review files that haven't changed but need inspection
- Focus review on a subset of changes
- Review legacy code for issues

**When to use code-review-changes:**
- Review all changes before committing
- Comprehensive pre-commit check
- Don't know which files need review

## Installation

This plugin should be placed in your Claude Code plugins directory. The command is automatically available when using Claude Code.

## Best Practices

### Using `/code-review-files`
- Specify files that are logically related for context
- Review both changed and unchanged files together when they interact
- Use for targeted review of critical code paths
- Run on files you're unfamiliar with before modifying them

### When to use
- Before modifying unfamiliar code
- After making changes to specific files
- When reviewing critical code paths
- To audit legacy code for issues
- When you want focused review (not all changes)

### When not to use
- To review all uncommitted changes (use `/code-review-changes`)
- For files outside the git repository
- For binary files or non-code files

## Workflow Integration

### Targeted pre-commit review:
```bash
# Make changes to specific files
vim src/feature.ts src/utils.ts

# Review only those files
/code-review-files src/feature.ts src/utils.ts

# Fix any issues found

# Commit when ready
git add -A
git commit -m "Add new feature"
```

### Legacy code audit:
```bash
# Review existing code before modifying
/code-review-files src/legacy/old-module.ts

# Understand issues before making changes
# Then proceed with modifications
```

### Mixed review (changed + unchanged):
```bash
# You've modified feature.ts which depends on utils.ts
# Review both, even though utils.ts hasn't changed
/code-review-files src/feature.ts src/utils.ts

# This reviews:
# - Changes in feature.ts
# - Full content of utils.ts (to check for integration issues)
```

## Requirements

- Git repository (must be inside a git repo)
- One or more valid file paths
- CLAUDE.md files (optional but recommended for guideline checking)

## Troubleshooting

### "Not a git repository"

**Issue**: The plugin reports not in a git repo

**Solution**:
- Make sure you're inside a git repository
- Check with `git rev-parse --git-dir`

### "No valid files specified"

**Issue**: No files found to review

**Solution**:
- Check file paths are correct
- Ensure files exist in the repository
- Use relative paths from the repository root

### Review takes too long

**Issue**: Agents are slow on large files

**Solution**:
- Normal for large files - agents run in parallel
- Consider reviewing fewer files at once
- Large unchanged files take longer than diffs

### Different results for changed vs unchanged files

**Issue**: More issues found in unchanged files

**Solution**:
- This is expected - full file review is more comprehensive
- Changed file review focuses only on the diff
- Unchanged file review checks entire content

## Tips

- **Review related files together**: Files that interact should be reviewed together
- **Use for code exploration**: Review unfamiliar files before modifying them
- **Combine with code-review-changes**: Use `/code-review-changes` for all changes, `/code-review-files` for focused review
- **Keep CLAUDE.md updated**: Better guidelines = better reviews
- **Trust the validation**: Multi-agent validation prevents noise

## Configuration

### Customizing output location

Use the `--output-file` flag to specify a custom location:
```bash
/code-review-files --output-file docs/reviews/feature-review.md src/feature.ts
```

### Customizing review focus

Edit `commands/code-review-files.md` to add or modify agent tasks:
- Add security-focused agents
- Add performance analysis agents
- Add accessibility checking agents
- Add documentation quality checks

## Technical Details

### Agent architecture
- **2x CLAUDE.md compliance agents**: Redundancy for guideline checks
- **1x bug detector**: Focused on bugs in reviewed content
- **1x logic/security analyzer**: Context-aware issue detection
- **Nx validation agents**: One per issue for independent confirmation

### Review modes
- **Changed files**: Reviews `git diff HEAD -- <file>` output
- **Unchanged files**: Reviews full file content via Read tool

### Git commands used
- `git diff HEAD -- <file>` - Get changes for specific file
- `git branch --show-current` - Current branch name
- `git rev-parse --git-dir` - Verify git repository

### Output locations
- Terminal: Formatted markdown displayed directly
- File: `.code-review-files.md` (default) or custom path

## Author

Jesse Naranjo

Based on the code-review plugin by Boris Cherny (boris@anthropic.com)

## Version

1.0.0
