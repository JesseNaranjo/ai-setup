# Settings Loader

Load and apply project-specific settings from `.claude/code-review.local.md`.

## Settings Timing

Settings are loaded at command execution time, not at Claude Code startup. Changes to `.claude/code-review.local.md` take effect immediately on the next review command invocation - no restart required.

## Settings File Location

```
project-root/
└── .claude/
    └── code-review.local.md    # Project-specific settings
```

## Loading Process

Before starting the review, check for settings:

### 1. Check for Settings File

Look for `.claude/code-review.local.md` in the project root.

If the file does not exist, use default settings and continue.

### 2. Parse YAML Frontmatter

If the file exists, parse the YAML frontmatter to extract settings:

```yaml
---
enabled: true
output_dir: "."
skip_agents: []
min_severity: "suggestion"
language: ""
additional_test_patterns: []
custom_rules: []
---
```

### 3. Check Enabled Flag

If `enabled: false`, stop and inform the user: "Code review plugin is disabled for this project. Edit .claude/code-review.local.md to enable."

### 4. Apply Settings

Apply settings to the review process:

| Setting | How to Apply |
|---------|--------------|
| `output_dir` | Prepend to default output filename (unless --output-file overrides) |
| `skip_agents` | Exclude listed agents from the review phases |
| `min_severity` | Filter output to only show issues at or above this severity |
| `language` | Use as language override (unless --language flag overrides) |
| `additional_test_patterns` | Merge with default test patterns when finding related tests |
| `custom_rules` | Pass to compliance-agent as additional rules to check |

### 5. Read Project Instructions

Read the markdown body (after the YAML frontmatter) and pass it to all review agents as additional context under "Project-Specific Instructions".

## Default Settings

When no settings file exists:

```yaml
enabled: true
output_dir: "."
skip_agents: []
min_severity: "suggestion"
language: ""  # auto-detect
additional_test_patterns: []
custom_rules: []
```

## Settings Priority

Command-line flags extend or override settings file:

1. `--output-file` overrides `output_dir`
2. `--language` overrides `language`
3. `--prompt` appends to project instructions (does not replace)

## Example Usage

### Skip Certain Agents

```yaml
---
skip_agents: ["architecture", "api-contracts"]
---
```

This project only needs bug, security, performance, compliance, error-handling, and test-coverage checks.

### Focus on Critical Issues Only

```yaml
---
min_severity: "major"
---
```

Only report Critical and Major issues; skip Minor and Suggestions.

### Custom Output Location

```yaml
---
output_dir: "./docs/reviews"
---
```

Save review files to `docs/reviews/` instead of project root.

### Add Custom Rules

```yaml
---
custom_rules:
  - pattern: "console\\.log"
    message: "Remove console.log statements before committing"
    severity: "minor"
  - pattern: "\\bany\\b"
    message: "Avoid using 'any' type - use specific types"
    severity: "suggestion"
---
```

### Provide Project Context

```yaml
---
enabled: true
---

# Project Context

This is a high-security financial application. Pay extra attention to:
- Input validation on all user-facing endpoints
- SQL injection in database queries
- Proper error handling that doesn't leak sensitive information

## Known Patterns

We use a custom ORM that sanitizes inputs automatically. Calls to `db.query()` are safe.
```

## Gitignore

Remind users to add to `.gitignore`:

```gitignore
# Claude Code plugin settings (user-specific)
.claude/*.local.md
```
