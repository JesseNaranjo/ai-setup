---
allowed-tools: Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git branch:*), Bash(git rev-parse:*), Bash(ls:*), Read, Write, Glob
description: Code review specific files - reviews changes if present, otherwise reviews entire file
---

Provide a comprehensive 10-agent code review for specific files. If a file has uncommitted changes, review those changes. If a file has no uncommitted changes, review the entire file.

**Required argument**: One or more file paths to review (space-separated)

Example usage:
- `/code-review-files src/utils.ts`
- `/code-review-files src/api/handler.ts src/models/user.ts`
- `/code-review-files --output-file review.md src/feature.ts`

---

## Step 1: Input Validation

Launch a Haiku agent to validate the input and check for changes:

1. Parse the arguments to extract:
   - File paths (everything except `--output-file <path>`)
   - Output file path (from `--output-file` flag, default: `.code-review-files.md`)

2. Check if the current directory is a git repository using `git rev-parse --git-dir`
   - If not a git repository, stop and inform the user: "Not a git repository."

3. Verify that each specified file exists (using `ls` or checking file paths)
   - If a file doesn't exist, note it and exclude from review

4. For each valid file, check if there are uncommitted changes using `git diff HEAD -- <file>`
   - Create two lists:
     - **Files with changes**: Files that have uncommitted changes
     - **Files without changes**: Files that have no uncommitted changes (review entire file)

5. If no valid files were specified, stop and inform the user: "No valid files specified for review."

---

## Step 2: Context Discovery (Enhanced)

Launch a Haiku agent to gather context:

1. Find all relevant CLAUDE.md and other AI Agent Instructions files:
   - The root CLAUDE.md file, if it exists
   - Any CLAUDE.md files in directories containing the specified files
   - Use `glob` to find CLAUDE.md files
   - The root .ai/AI-AGENT-INSTRUCTIONS.md file, if it exists
   - Use `glob` to find AI-AGENT-INSTRUCTIONS.md files
   - The root .github/copilot-instructions.md file, if it exists
   - Use `glob` to find copilot-instructions.md files

2. Detect project type:
   - Check for `package.json` in repository root (Node.js/TypeScript project)
   - Check for `*.csproj`, `*.sln`, or `*.slnx` files (.NET project)
   - Projects can be both (mixed)

3. Find related test files for each file being reviewed:
   - **Node.js**: Look for `*.test.ts`, `*.spec.ts`, `*.test.js`, `*.spec.js`, `__tests__/` patterns
   - **.NET**: Look for `*.Tests.cs`, `*Tests/` projects, `*.UnitTests/` patterns
   - **Common**: Look for a `tests/` directory at the project root or inside `src/`

4. Return:
   - List of CLAUDE.md and other AI Agent Instructions file paths
   - Detected project type(s)
   - List of related test file paths

---

## Step 3: Content Gathering (Enhanced)

Launch a Sonnet agent to gather the content to review:

1. Run `git branch --show-current` to get the current branch name

2. For files with changes:
   - Run `git diff HEAD -- <file>` to get the diff
   - Also read the full file content for broader context

3. For files without changes:
   - Read the entire file content

4. Read related test files (from Step 2) for context

5. Summarize what will be reviewed:
   - Which files have changes being reviewed
   - Which files are being reviewed in their entirety (no changes)
   - Detected project type(s)
   - Related test files found

---

## Step 4: 10-Agent Parallel Review

Follow the agent definitions from `shared/review-workflow.md`.

Launch ALL 10 agents in parallel:

| Agent | Model | Category | Focus |
|-------|-------|----------|-------|
| 1 | Opus | CLAUDE.md and other AI Agent Instructions Compliance | Standards adherence |
| 2 | Opus | CLAUDE.md and other AI Agent Instructions Compliance | Independent verification |
| 3 | Opus | Bug Detection | Logical errors, null refs, off-by-one |
| 4 | Opus | Bug Detection | Edge cases, race conditions, state |
| 5 | Opus | Security | Injection, auth, secrets, OWASP |
| 6 | Opus | Performance | Complexity, memory, hot paths, N+1 |
| 7 | Sonnet | Architecture | Coupling, patterns, SOLID |
| 8 | Sonnet | API & Contracts | Breaking changes, compatibility |
| 9 | Sonnet | Error Handling | Try/catch gaps, resilience |
| 10 | Sonnet | Test Coverage | Missing tests, test suggestions |

Each agent receives:
- The current branch name
- A summary of what is being reviewed
- Which files have changes vs which are being reviewed in full
- The detected project type (for language-specific focus)
- The CLAUDE.md and other AI Agent Instructions files relevant to each file being reviewed
- For files with changes: both the diff AND full file content
- For files without changes: the full file content
- Related test files for context

Each agent returns issues with:
- Issue title
- File path and line range
- Description
- Category
- Suggested severity (Critical/Major/Minor/Suggestion)

**CRITICAL: We only want HIGH SIGNAL issues.** See `shared/review-workflow.md` for details.

---

## Step 5: Universal Validation

For EACH issue found in Step 4 by ALL agents, launch validation subagents:

**Validation model assignment:**
- Opus validators: Bugs (agents 3-4), Security (agent 5), Performance (agent 6)
- Sonnet validators: CLAUDE.md and other AI Agent Instructions (agents 1-2), Architecture (agent 7), API (agent 8), Errors (agent 9), Tests (agent 10)

Each validator:
1. Reviews the issue description
2. Reads the relevant file(s) to verify the issue exists
3. Checks if this is actually a problem or a false positive
4. Verifies severity classification is appropriate
5. Returns: VALID, INVALID, or DOWNGRADE (with new severity)

---

## Step 6: Aggregation

1. Filter out any issues marked INVALID in Step 5
2. Apply severity downgrades where indicated
3. Deduplicate issues by file+line range (merge when multiple agents flag same issue)
4. Add consensus badges for issues flagged by 2+ agents

---

## Step 7: Generate Output

Generate the review output following the format in `shared/review-workflow.md`.

If NO issues were found:
```markdown
## Code Review

**Reviewed:** [N] file(s) | **Branch:** [branch-name]
**Review Depth:** Comprehensive (10-agent analysis)

No issues found. All checks passed:
- CLAUDE.md and other AI Agent Instructions compliance
- Bug detection (logical errors, edge cases)
- Security analysis
- Performance review
- Architecture patterns
- API compatibility
- Error handling
- Test coverage

Files reviewed:
- With changes: [list or "none"]
- Full file review: [list or "none"]
```

If issues WERE found, use the full summary table format from `shared/review-workflow.md`.

---

## Step 8: Write Output

1. Display the formatted review in the terminal

2. Write the same content to a file:
   - Default: `.code-review-files.md` in the repository root
   - If `--output-file <path>` argument was provided, use that path instead

3. At the end, print: "Review saved to: [filepath]"

**Suggestion formatting:**
- For small fixes (up to 5 lines changed), include a committable suggestion in a code block
- **Suggestions must be COMPLETE.** If a fix requires additional changes elsewhere, do NOT use a suggestion block
- For larger fixes (6+ lines, structural changes, or changes spanning multiple locations), include a copyable prompt:
  ```
  Fix [file:line]: [brief description of issue and suggested fix]
  ```

**IMPORTANT: Only report ONE entry per unique issue. Do not duplicate issues.**

---

## False Positives (Do NOT flag)

- Pre-existing issues in files with changes (issues that exist before these changes)
- Something that appears to be a bug but is actually correct
- Pedantic nitpicks that a senior engineer would not flag
- Issues that a linter will catch (do not run the linter to verify)
- General code quality concerns (e.g., lack of test coverage) unless explicitly required in CLAUDE.md and other AI Agent Instructions
- Issues mentioned in CLAUDE.md and other AI Agent Instructions but explicitly silenced in the code (e.g., via a lint ignore comment)

---

## Notes

- Use git CLI to interact with the repository. Do not use GitHub CLI.
- Create a todo list before starting.
- You must cite each issue with its file path and line numbers (e.g., `src/utils.ts:42-48`).
- When referencing CLAUDE.md and other AI Agent Instructions rules, quote the exact rule being violated.
- If no issues are found, output the "no issues" format shown in step 7.
- File paths should be relative to the repository root.
- Line numbers should reference the lines in the actual file (not the diff line numbers).
- When reviewing full files (no changes), be more lenient - focus on clear bugs, not style issues.
