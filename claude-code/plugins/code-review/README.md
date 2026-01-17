# Code Review Plugin

Comprehensive 10-agent code review for Node.js and .NET projects using Claude's multi-agent architecture.

## Overview

The Code Review Plugin provides automated, in-depth code review using 10 specialized agents that analyze code from different perspectives. Each agent focuses on a specific aspect of code quality, and their findings are validated and aggregated to produce high-signal, actionable feedback.

### Key Features

- **10-Agent Architecture**: Comprehensive coverage across multiple dimensions
- **Language-Aware**: Specialized checks for Node.js/TypeScript and .NET/C# projects
- **Validation Layer**: Every issue is independently validated to reduce false positives
- **Severity Classification**: Issues categorized as Critical, Major, Minor, or Suggestion
- **Consensus Scoring**: Issues flagged by multiple agents get higher confidence
- **Deduplication**: Identical issues from multiple agents are merged

## Commands

### `/code-review-files`

Reviews specific files you specify. Intelligently handles files with and without uncommitted changes.

**Usage:**
```bash
/code-review-files <file1> [file2] [...] [--output-file <path>]
```

**Arguments:**
- `<file>`: One or more file paths to review (required)
- `--output-file <path>`: Custom output location (default: `.code-review-files.md`)

**Examples:**
```bash
# Review a single file
/code-review-files src/utils.ts

# Review multiple files
/code-review-files src/api/handler.ts src/models/user.ts

# Review with custom output
/code-review-files --output-file reviews/feature.md src/feature.ts
```

**Behavior:**
- Files with uncommitted changes: Reviews the diff + full file context
- Files without changes: Reviews the entire file content

### `/code-review-staged`

Reviews all staged git changes (files added with `git add`).

**Usage:**
```bash
/code-review-staged [--output-file <path>]
```

**Arguments:**
- `--output-file <path>`: Custom output location (default: `.code-review-staged.md`)

**Examples:**
```bash
# Review all staged changes
git add src/feature.ts
/code-review-staged

# With custom output
/code-review-staged --output-file reviews/pre-commit.md
```

## Review Dimensions

The 10-agent architecture covers these review dimensions:

| Agent | Model | Category | Focus |
|-------|-------|----------|-------|
| 1-2 | Opus | AI Agent Instructions Compliance | Standards adherence, rule verification |
| 3 | Opus | Bug Detection | Runtime bugs, null refs, off-by-one errors |
| 4 | Opus | Bug Detection | Edge cases, race conditions, state issues |
| 5 | Opus | Security | Injection, auth, secrets, OWASP vulnerabilities |
| 6 | Opus | Performance | Complexity, memory, hot paths, N+1 queries |
| 7 | Sonnet | Architecture | Coupling, patterns, SOLID principles |
| 8 | Sonnet | API & Contracts | Breaking changes, compatibility issues |
| 9 | Sonnet | Error Handling | Try/catch gaps, resilience, recovery |
| 10 | Sonnet | Test Coverage | Missing tests, test recommendations |

## Severity Levels

| Severity | Definition | Action |
|----------|------------|--------|
| **Critical** | Data loss, security breach, production outage risk | Must fix before merge |
| **Major** | Significant bug or violation affecting functionality | Should fix before merge |
| **Minor** | Small issue that should be addressed | Can merge, fix soon |
| **Suggestion** | Recommended improvement for future consideration | Optional |

## Language Support

### Node.js / TypeScript

Detected by presence of `package.json`.

**Language-specific checks:**
- Unhandled promise rejections
- Async/await pitfalls
- `this` binding issues
- Prototype pollution
- ReDoS vulnerabilities
- Event loop blocking
- Memory leaks from closures
- CommonJS vs ESM issues
- React hooks violations
- Test file detection: `*.test.ts`, `*.spec.ts`, `__tests__/`

### .NET / C#

Detected by presence of `*.csproj` or `*.sln` files.

**Language-specific checks:**
- Null reference exceptions
- `IDisposable` not disposed
- Async deadlocks (`Result`/`Wait()`)
- SQL injection via string concatenation
- Missing `[Authorize]` attributes
- N+1 Entity Framework queries
- Boxing/unboxing overhead
- DI anti-patterns
- Missing `ConfigureAwait(false)`
- Test file detection: `*.Tests.cs`, `*Tests/` projects

## Output Format

### Summary Table

```markdown
## Code Review

**Reviewed:** 5 file(s) | **Branch:** feature/new-auth
**Review Depth:** Comprehensive (10-agent analysis)

### Summary

| Category | Critical | Major | Minor | Suggestions |
|----------|----------|-------|-------|-------------|
| AI Agent Instructions | 0 | 1 | 0 | 0 |
| Bugs | 0 | 2 | 1 | 0 |
| Security | 1 | 0 | 0 | 0 |
| Performance | 0 | 0 | 1 | 2 |
| ...
```

### Issue Format

```markdown
**1. SQL Injection Vulnerability** `Critical` `Security` [2 agents]
`src/data/UserRepository.cs:45-48`

The query uses string concatenation with user input, allowing SQL injection.

```suggestion
var query = "SELECT * FROM Users WHERE Id = @id";
cmd.Parameters.AddWithValue("@id", userId);
```
```

## Workflow Integration

### Pre-commit Review

```bash
# Make changes
vim src/feature.ts

# Stage and review
git add src/feature.ts
/code-review-staged

# Fix issues, then commit
git commit -m "Add new feature"
```

### Targeted File Review

```bash
# Review specific files before modifying
/code-review-files src/legacy/old-module.ts

# Review related files together
/code-review-files src/api/handler.ts src/models/user.ts src/services/auth.ts
```

### Legacy Code Audit

```bash
# Review critical paths before changes
/code-review-files src/payments/processor.ts src/payments/validator.ts
```

## Best Practices

1. **Review incrementally**: Smaller changesets produce more focused reviews
2. **Maintain CLAUDE.md and other AI Agent Instructions**: Clear guidelines improve compliance checking
3. **Review related files together**: Context helps catch integration issues
4. **Address Critical/Major first**: Focus on high-impact issues
5. **Trust validation**: Multi-agent validation filters most false positives

## Requirements

- Git repository
- Claude Code CLI
- For `/code-review-staged`: Staged changes (use `git add` first)
- For `/code-review-files`: Valid file paths

## Technical Details

### Workflow Steps

1. **Input Validation**: Verify git repo, files/staged changes exist
2. **Context Discovery**: Find CLAUDE.md and other AI Agent Instructions files, detect project type, find test files
3. **Content Gathering**: Collect diffs, full files, and related context
4. **10-Agent Review**: Parallel analysis across all dimensions
5. **Validation**: Independent verification of each issue
6. **Aggregation**: Deduplication, consensus scoring, severity adjustment
7. **Output**: Formatted markdown to terminal and file

### File Structure

```
code-review/
├── .claude-plugin/
│   └── plugin.json           # Plugin metadata
├── commands/
│   ├── code-review-files.md  # /code-review-files command
│   └── code-review-staged.md # /code-review-staged command
├── shared/
│   └── review-workflow.md    # Shared 10-agent workflow
└── README.md                 # This file
```

## Troubleshooting

### "No staged changes to review"
- Run `git add` to stage changes first
- Check `git status` to verify staged files

### "Not a git repository"
- Ensure you're inside a git repository
- Run `git rev-parse --git-dir` to verify

### "No valid files specified"
- Check file paths are correct
- Use relative paths from repository root

### Too many issues
- Focus on Critical and Major severity first
- Consider reviewing smaller changesets
- Update CLAUDE.md and other AI Agent Instructions to clarify acceptable patterns

### Missing language-specific checks
- Verify `package.json` exists for Node.js detection
- Verify `*.csproj` or `*.sln` exists for .NET detection

## Author

Jesse Naranjo

Based on the code-review plugin by Boris Cherny (boris@anthropic.com)

## Version

2.0.0
