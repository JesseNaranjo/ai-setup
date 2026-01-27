# Scope Determination Details

Detailed procedures for determining review scope. Referenced by skill workflow step 1.

## Scope Options

| Option | Description | When to Use |
|--------|-------------|-------------|
| **Staged changes** | Review `git diff --cached` | Pre-commit review |
| **Specific files** | User provides file paths | Targeted review |
| **Directory** | All files in a directory | Module-level audit |

## Decision Points

### Staged Changes

1. Run `git diff --cached --name-only` to get file list
2. If no staged changes exist, inform user and offer alternatives
3. Get full diff with `git diff --cached`

### Specific Files

1. Verify each file exists using Glob
2. If file not found, ask for correction
3. Read each file's content

### Directory

1. Use Glob to find all code files (`**/*.ts`, `**/*.js`, `**/*.cs`, etc.)
2. Warn if more than 20 files (suggest narrowing scope)
3. Prioritize files relevant to the review type

## Edge Cases

- **Empty file list:** "No files found to review. Please check the path or stage your changes."
- **Binary files:** Skip and note in output
- **Very large files (>5000 lines):** Warn user, suggest focusing on specific functions

## Clarifying User Intent

When the user's request is ambiguous, use AskUserQuestion to clarify:

**Example questions:**
- "Do you want to review staged changes or specific files?"
- "This directory contains 45 files. Should I review all of them or focus on a specific subdirectory?"
- "I found both TypeScript and C# files. Should I review both or focus on one language?"
