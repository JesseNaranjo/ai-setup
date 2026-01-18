# Output Generation

Shared output generation and file writing logic for all review commands.

## Process

### 1. Generate Review Output

Follow the format in `shared/output-format.md`:

1. **Header**: Use the review depth description provided by the command
2. **Summary Table**: Include only the categories that were reviewed
3. **Issues**: Group by severity (Critical, Major, Minor, Suggestions)
4. **Cross-Cutting Insights**: Include if synthesis agents produced insights
5. **Test Recommendations**: Include if applicable

### 2. Display Output

Display the formatted review in the terminal.

### 3. Write to File

Write the same content to a file:

| Command | Default Output File |
|---------|---------------------|
| `/deep-review` | `.deep-review.md` |
| `/deep-review-staged` | `.deep-review-staged.md` |
| `/quick-review` | `.quick-review.md` |
| `/quick-review-staged` | `.quick-review-staged.md` |

If `--output-file <path>` argument was provided, use that path instead.

### 4. Confirm Output

At the end, print: "Review saved to: [filepath]"

## Fix Formatting Rules

See `shared/output-format.md` for complete details:

- **fix_type: diff** for single-location fixes ≤10 lines
  - Show inline diff block with `-` and `+` lines
  - Must be complete drop-in replacements (no "..." or partial code)

- **fix_type: prompt** for multi-location, structural, or complex fixes
  - Show copyable Claude Code prompt in blockquote format
  - Must be specific and actionable

## Important

**Only report ONE entry per unique issue. Do not duplicate issues.**
