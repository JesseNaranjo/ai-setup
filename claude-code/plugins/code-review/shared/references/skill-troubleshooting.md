# Skill Troubleshooting

Common issues and solutions when using review skills.

## Agent Returns No Issues But Problems Exist

**Symptoms:** Review completes but expected issues aren't flagged.

**Checklist:**
- Verify file content is included in the prompt
- Check that diffs show the problematic lines
- Ensure project_type is correctly detected
- Try expanding scope to include related files

**Solutions:**
1. Re-run with explicit file paths instead of staged changes
2. Check if file is in .gitignore (won't appear in staged)
3. Verify the issue is in changed lines (not pre-existing)

## Too Many False Positives

**Symptoms:** Review flags many issues that aren't real problems.

**Checklist:**
- Are issues in test files? (often intentional)
- Are issues in commented code?
- Are "vulnerabilities" in internal-only code?

**Solutions:**
1. Increase min_severity to "Major" in settings
2. Exclude test files if not relevant
3. Add context about intentional exceptions in project settings

## Agent Times Out

**Symptoms:** Review doesn't complete, hangs indefinitely.

**Checklist:**
- How many files are being reviewed?
- How large are the files?
- Is the diff very large?

**Solutions:**
1. Reduce scope to fewer files
2. Split large files into focused reviews
3. Use quick mode for initial scan, then deep review specific areas

## Wrong Language Detected

**Symptoms:** Node.js checks on C# code or vice versa.

**Checklist:**
- Is there both package.json and .csproj in the repo?
- Are files in a subdirectory with different project type?

**Solutions:**
1. Use `--language nodejs` or `--language dotnet` flag
2. Set `language:` in `.claude/code-review.local.md`
3. Run review from the specific project subdirectory

## Skill Not Triggering

**Symptoms:** Asking for "security review" doesn't invoke the skill.

**Checklist:**
- Is the plugin installed and enabled?
- Are you using recognized trigger phrases?

**Solutions:**
1. Use exact trigger phrases: "security review", "check for vulnerabilities"
2. Verify plugin is listed in `/help`
3. Restart Claude Code to reload plugins

## Cross-Cutting Issues Missed

**Symptoms:** Individual category reviews pass but integration issues exist.

**Solutions:**
1. Use `/deep-code-review` which includes synthesis phase
2. Explicitly ask: "Check if security fixes affect performance"
3. Review related files together, not in isolation
