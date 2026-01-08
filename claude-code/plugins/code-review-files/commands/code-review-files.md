---
allowed-tools: Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git branch:*), Bash(git rev-parse:*), Read, Write, Glob
description: Code review specific files - reviews changes if present, otherwise reviews entire file
---

Provide a code review for specific files. If a file has uncommitted changes, review those changes. If a file has no uncommitted changes, review the entire file.

**Required argument**: One or more file paths to review (space-separated)

Example usage:
- `/code-review-files src/utils.ts`
- `/code-review-files src/api/handler.ts src/models/user.ts`
- `/code-review-files --output-file review.md src/feature.ts`

To do this, follow these steps precisely:

1. Launch a haiku agent to validate the input and check for changes:
   - Parse the arguments to extract file paths (everything except `--output-file <path>`)
   - Check if the current directory is a git repository using `git rev-parse --git-dir`
   - If not a git repository, stop and inform the user: "Not a git repository."
   - Verify that each specified file exists (using `ls` or checking file paths)
   - For each file, check if there are uncommitted changes using `git diff HEAD -- <file>`
   - Create two lists:
     - **Files with changes**: Files that have uncommitted changes
     - **Files without changes**: Files that have no uncommitted changes (review entire file)
   - If no valid files were specified, stop and inform the user: "No valid files specified for review."

2. Launch a haiku agent to return a list of file paths (not their contents) for all relevant CLAUDE.md files including:
   - The root CLAUDE.md file, if it exists
   - Any CLAUDE.md files in directories containing the specified files
   - Use `glob` to find CLAUDE.md files

3. Launch a sonnet agent to gather the content to review:
   - Run `git branch --show-current` to get the current branch name
   - For files with changes: Run `git diff HEAD -- <file>` to get the diff
   - For files without changes: Read the entire file content
   - Summarize what will be reviewed:
     - Which files have changes being reviewed
     - Which files are being reviewed in their entirety (no changes)

4. Launch 4 agents in parallel to independently review the content. Each agent should return the list of issues, where each issue includes a description and the reason it was flagged (e.g. "CLAUDE.md adherence", "bug"). The agents should do the following:

   Agents 1 + 2: CLAUDE.md compliance sonnet agents
   Audit the content for CLAUDE.md compliance in parallel. Note: When evaluating CLAUDE.md compliance for a file, you should only consider CLAUDE.md files that share a file path with the file or parents.

   Agent 3: Opus bug agent (parallel subagent with agent 4)
   Scan for obvious bugs. For files with changes, focus only on the diff. For files without changes, review the entire file. Flag only significant bugs; ignore nitpicks and likely false positives.

   Agent 4: Opus bug agent (parallel subagent with agent 3)
   Look for problems in the code being reviewed. This could be security issues, incorrect logic, etc. For files with changes, only look at the changed code. For files without changes, review the entire file.

   **CRITICAL: We only want HIGH SIGNAL issues.** This means:
   - Objective bugs that will cause incorrect behavior at runtime
   - Clear, unambiguous CLAUDE.md violations where you can quote the exact rule being broken

   We do NOT want:
   - Subjective concerns or "suggestions"
   - Style preferences not explicitly required by CLAUDE.md
   - Potential issues that "might" be problems
   - Anything requiring interpretation or judgment calls

   If you are not certain an issue is real, do not flag it. False positives erode trust and waste reviewer time.

   In addition to the above, each subagent should be told:
   - The current branch name
   - A summary of what is being reviewed
   - Which files have changes vs which are being reviewed in full
   This will help provide context regarding the author's intent.

5. For each issue found in the previous step by agents 3 and 4, launch parallel subagents to validate the issue. These subagents should get the branch name and review summary along with a description of the issue. The agent's job is to review the issue to validate that the stated issue is truly an issue with high confidence. For example, if an issue such as "variable is not defined" was flagged, the subagent's job would be to validate that is actually true in the code. Another example would be CLAUDE.md issues. The agent should validate that the CLAUDE.md rule that was violated is scoped for this file and is actually violated. Use Opus subagents for bugs and logic issues, and sonnet agents for CLAUDE.md violations.

6. Filter out any issues that were not validated in step 5. This step will give us our list of high signal issues for our review.

7. Generate the review output. The output should be written to BOTH the terminal AND a markdown file.

   If NO issues were found, output:
   ```
   ## File Code Review

   No issues found. Checked for bugs and CLAUDE.md compliance.

   Files reviewed:
   - With changes: [list of files with changes, or "none"]
   - Full file review: [list of files reviewed entirely, or "none"]
   ```

   If issues WERE found, format the output as:
   ```
   ## File Code Review

   Reviewed [N] file(s):
   - With changes: [list or "none"]
   - Full file review: [list or "none"]

   ### Issues Found: [count]

   **1. [Issue title]** ([Category: Bug/CLAUDE.md violation])
   `path/to/file.ts:line-range`

   [Description of the issue]

   ```suggestion
   [corrected code here - only for small fixes up to 5 lines]
   ```

   **2. [Next issue]** ...
   ```

8. Write the review output:
   - Display the formatted review in the terminal
   - Write the same content to a file:
     - Default: `.code-review-files.md` in the repository root
     - If `--output-file <path>` argument was provided, use that path instead
   - At the end, print: "Review saved to: [filepath]"

   For suggestions:
   - For small fixes (up to 5 lines changed), include a committable suggestion in a code block
   - **Suggestions must be COMPLETE.** If a fix requires additional changes elsewhere (e.g., renaming a variable requires updating all usages), do NOT use a suggestion block.
   - For larger fixes (6+ lines, structural changes, or changes spanning multiple locations), do NOT use suggestion blocks. Instead:
     1. Describe what the issue is
     2. Explain the suggested fix at a high level
     3. Include a copyable prompt for Claude Code that the user can use to fix the issue:
        ```
        Fix [file:line]: [brief description of issue and suggested fix]
        ```

   **IMPORTANT: Only report ONE entry per unique issue. Do not duplicate issues.**

Use this list when evaluating issues in Steps 4 and 5 (these are false positives, do NOT flag):

- Pre-existing issues in files with changes (issues that exist in the codebase before these changes)
- Something that appears to be a bug but is actually correct
- Pedantic nitpicks that a senior engineer would not flag
- Issues that a linter will catch (do not run the linter to verify)
- General code quality concerns (e.g., lack of test coverage, general security issues) unless explicitly required in CLAUDE.md
- Issues mentioned in CLAUDE.md but explicitly silenced in the code (e.g., via a lint ignore comment)

Notes:

- Use git CLI to interact with the repository. Do not use GitHub CLI.
- Create a todo list before starting.
- You must cite each issue with its file path and line numbers (e.g., `src/utils.ts:42-48`).
- When referencing CLAUDE.md rules, quote the exact rule being violated.
- If no issues are found, output the "no issues" format shown in step 7.
- File paths should be relative to the repository root.
- Line numbers should reference the lines in the actual file (not the diff line numbers).
- When reviewing full files (no changes), be more lenient - focus on clear bugs, not style issues.
