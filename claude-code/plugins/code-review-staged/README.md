# Code Review Staged Plugin

Automated code review for staged git changes using multiple specialized agents with confidence-based scoring to filter false positives.

## Overview

The Code Review Staged Plugin automates code review for staged changes in your git repository. It launches multiple agents in parallel to independently audit changes from different perspectives, using confidence scoring to filter out false positives and ensure only high-quality, actionable feedback is reported.

## Commands

### `/code-review-staged`

Performs automated code review on staged git changes (only changes added with `git add`) using multiple specialized agents.

**What it does:**
1. Checks if there are any staged changes to review
2. Gathers relevant CLAUDE.md guideline files from the repository
3. Summarizes the staged changes
4. Launches 4 parallel agents to independently review:
   - **Agents #1 & #2**: Audit for CLAUDE.md compliance
   - **Agent #3**: Scan for obvious bugs in changes
   - **Agent #4**: Analyze for security issues and incorrect logic
5. Validates each issue with follow-up agents
6. Filters out low-confidence issues
7. Outputs review to terminal AND saves to markdown file

**Usage:**
```bash
/code-review-staged [--output-file <path>]
```

**Options:**
- `--output-file <path>`: Save the review to a custom file path (default: `.code-review-staged.md` in repo root)

**Example workflow:**
```bash
# Make some changes to your code
# Stage the changes you want to review:
git add src/feature.ts

# Run code review:
/code-review-staged

# Claude will:
# - Check for staged changes
# - Launch 4 review agents in parallel
# - Validate each issue found
# - Output issues to terminal
# - Save review to .code-review-staged.md

# Use custom output file:
/code-review-staged --output-file reviews/my-review.md
```

**Features:**
- Reviews only staged changes (ignores unstaged and untracked files)
- Multiple independent agents for comprehensive review
- Confidence-based validation reduces false positives
- CLAUDE.md compliance checking with explicit guideline verification
- Bug detection focused on changes (not pre-existing issues)
- Outputs to both terminal and file for reference
- File path references with line numbers

**Review output format:**
```markdown
## Code Review

Reviewed staged changes (3 files modified)

### Issues Found: 2

**1. Missing error handling** (Bug)
`src/utils.ts:42-45`

The async function doesn't handle promise rejection...

```suggestion
try {
  await fetchData();
} catch (error) {
  console.error('Failed to fetch:', error);
}
```

**2. CLAUDE.md violation** (CLAUDE.md violation)
`src/api/handler.ts:78`

Violates rule: "Always validate input parameters"
...

---
Review saved to: .code-review-staged.md
```

**False positives filtered:**
- Pre-existing issues not introduced in changes
- Code that looks like a bug but isn't
- Pedantic nitpicks
- Issues linters will catch
- General quality issues (unless in CLAUDE.md)
- Issues with lint ignore comments

## Installation

This plugin should be placed in your Claude Code plugins directory. The command is automatically available when using Claude Code.

## Best Practices

### Using `/code-review-staged`
- Maintain clear CLAUDE.md files for better compliance checking
- Stage changes with `git add` before running the review
- Review agent findings as a starting point for self-review
- Update CLAUDE.md based on recurring review patterns

### When to use
- Before committing changes
- After staging significant modifications
- When working on critical code paths
- To validate changes before creating a PR

### When not to use
- When there are no staged changes (automatically detected)
- For trivial whitespace-only changes
- When you need to review already-committed code (use git diff ranges instead)

## Workflow Integration

### Pre-commit review workflow:
```bash
# Make your changes
vim src/feature.ts

# Stage changes for review
git add src/feature.ts

# Review before committing
/code-review-staged

# Review the automated feedback
# Make any necessary fixes

# Commit when ready
git commit -m "Add new feature"
```

### Review specific changes:
```bash
# Stage only the files you want to review
git add src/specific-file.ts

# Run review (will review only staged changes)
/code-review-staged

# Commit reviewed changes
git commit -m "Reviewed changes"
```

## Requirements

- Git repository (must be inside a git repo)
- Staged changes to review (use `git add` first)
- CLAUDE.md files (optional but recommended for guideline checking)

## Troubleshooting

### "No staged changes to review"

**Issue**: The plugin reports no changes

**Solution**:
- Run `git add` to stage changes before running the review
- Check `git status` to see if you have staged changes
- Make sure you're in a git repository

### Review takes too long

**Issue**: Agents are slow on large changesets

**Solution**:
- Normal for large changes - agents run in parallel
- 4 independent agents ensure thoroughness
- Consider staging smaller batches of changes

### Too many false positives

**Issue**: Review flags issues that aren't real

**Solution**:
- Validation agents should filter most false positives
- Make CLAUDE.md more specific about what matters
- Consider if the flagged issue is actually valid

### Output file not created

**Issue**: Review runs but no file is saved

**Solution**:
- Check write permissions in the repository
- Verify the output path is valid
- Check for any error messages during output

## Tips

- **Write specific CLAUDE.md files**: Clear guidelines = better reviews
- **Review incrementally**: Smaller staged changes = faster, more focused reviews
- **Use the output file**: Keep `.code-review-staged.md` for reference before committing
- **Iterate on guidelines**: Update CLAUDE.md based on patterns
- **Trust the validation**: Multi-agent validation prevents noise

## Configuration

### Customizing output location

Use the `--output-file` flag to specify a custom location:
```bash
/code-review-staged --output-file docs/reviews/feature-review.md
```

### Customizing review focus

Edit `commands/code-review-staged.md` to add or modify agent tasks:
- Add security-focused agents
- Add performance analysis agents
- Add accessibility checking agents
- Add documentation quality checks

## Technical Details

### Agent architecture
- **2x CLAUDE.md compliance agents**: Redundancy for guideline checks
- **1x bug detector**: Focused on obvious bugs in changes only
- **1x logic/security analyzer**: Context-aware issue detection
- **Nx validation agents**: One per issue for independent confirmation

### Git commands used
- `git diff --cached` - Get all staged changes
- `git diff --cached --stat` - Summary of staged changes
- `git diff --cached --name-only` - List of staged files
- `git branch --show-current` - Current branch name
- `git rev-parse --git-dir` - Verify git repository

### Output locations
- Terminal: Formatted markdown displayed directly
- File: `.code-review-staged.md` (default) or custom path

## Author

Jesse Naranjo

Based on the code-review plugin by Boris Cherny (boris@anthropic.com)

## Version

1.0.0
