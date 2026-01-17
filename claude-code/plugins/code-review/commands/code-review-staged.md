---
allowed-tools: Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git branch:*), Bash(git rev-parse:*), Read, Write, Glob
description: Code review staged git changes
---

Provide a comprehensive 10-agent code review for staged git changes.

Example usage:
- `/code-review-staged`
- `/code-review-staged --output-file review.md`

---

## Step 1: Staged Changes Validation

Launch a Haiku agent to check if there are any staged changes:

1. Check if the current directory is a git repository using `git rev-parse --git-dir`
   - If not a git repository, stop and inform the user: "Not a git repository."

2. Run `git diff --cached --stat` to see all staged changes
   - If there are no changes, stop and inform the user: "No staged changes to review."

3. Run `git diff --cached --name-only` to get the list of staged files

4. Parse any `--output-file <path>` argument (default: `.code-review-staged.md`)

---

## Step 2: Context Discovery (Enhanced)

Launch a Haiku agent to gather context:

1. Find all relevant CLAUDE.md and other AI Agent Instructions files:
   - The root CLAUDE.md file, if it exists
   - Any CLAUDE.md files in directories containing staged files
   - Use `glob` to find CLAUDE.md files
   - The root .ai/AI-AGENT-INSTRUCTIONS.md file, if it exists
   - Use `glob` to find AI-AGENT-INSTRUCTIONS.md files
   - The root .github/copilot-instructions.md file, if it exists
   - Use `glob` to find copilot-instructions.md files

2. Detect project type:
   - Check for `package.json` in repository root (Node.js/TypeScript project)
   - Check for `*.csproj`, `*.sln`, or `*.slnx` files (.NET project)
   - Projects can be both (mixed)

3. Find related test files for each staged file:
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

1. Run `git diff --cached` to get the full staged diff

2. Run `git branch --show-current` to get the current branch name

3. For each staged file, also read the full file content for broader context

4. Read related test files (from Step 2) for context

5. Summarize what changes are being made:
   - Number of files modified
   - Brief description of the changes
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
- A summary of the staged changes
- The staged diff
- Full file content for broader context
- The detected project type (for language-specific focus)
- The CLAUDE.md and other AI Agent Instructions files relevant to each staged file
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

Files reviewed: [list of staged files]
```

If issues WERE found, use the full summary table format from `shared/review-workflow.md`.

---

## Step 8: Write Output

1. Display the formatted review in the terminal

2. Write the same content to a file:
   - Default: `.code-review-staged.md` in the repository root
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

- Pre-existing issues (issues that exist in the codebase before these changes)
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
- Line numbers should reference the lines in the working copy (not the diff line numbers).
