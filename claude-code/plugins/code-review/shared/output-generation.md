# Output Generation

Shared output generation and file writing logic for all review commands.

## Process

### 0. Generate Usage Summary

Before generating the Code Review output, generate the Usage Summary section:

1. **Read tracking data**: Access the usage_tracking structure maintained during workflow execution
2. **Calculate totals**:
   - Total duration = review ended_at - review started_at
   - Agents invoked = count of agents with status "completed" or "failed"
   - Planned agents = total expected minus skipped agents
3. **Calculate phase metrics**:
   - Phase duration = phase ended_at - phase started_at
   - Agents completed = count per phase
   - Total findings = sum of findings_count per agent
4. **Record findings_count**: For each agent, count the number of issues in their output and record as `findings_count`
5. **Detect timing anomalies**:
   - Opus agents: < 15s = too fast `[!]`, > 180s = too slow `[*]`
   - Sonnet agents: < 10s = too fast `[!]`, > 120s = too slow `[*]`
   - Synthesis agents: < 5s = too fast `[!]`, > 90s = too slow `[*]`
6. **Format output**: Follow `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md` Usage Summary Section format

**Output order:**
1. Usage Summary (this step)
2. Code Review header and content (next step)

See `${CLAUDE_PLUGIN_ROOT}/shared/usage-tracking.md` for the tracking schema and `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md` for the output format.

### 1. Generate Review Output

Follow the format in `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md`:

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

See `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md` for complete details:

- **fix_type: diff** for single-location fixes ≤10 lines
  - Show inline diff block with `-` and `+` lines
  - Must be complete drop-in replacements (no "..." or partial code)

- **fix_type: prompt** for multi-location, structural, or complex fixes
  - Show copyable Claude Code prompt in blockquote format
  - Must be specific and actionable

## Important

**Only report ONE entry per unique issue. Do not duplicate issues.**
