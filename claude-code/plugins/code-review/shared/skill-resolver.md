# Skill Resolver

Resolve skill names to SKILL.md files and extract methodology content for embedding in agent prompts.

## Resolution Algorithm

For each skill in the `--skills` argument:

### 1. Parse Skill Name

Split on comma to get individual skill names.

### 2. Resolve to File Path

**If skill contains ":" (external plugin):**

Claude native install path structure (where `$HOME` is the user's home directory):
```
$HOME/.claude/plugins/cache/{source}/{plugin_name}/{version}/skills/{skill_name}/SKILL.md
```

Example:
```
$HOME/.claude/plugins/cache/claude-plugins-official/superpowers/4.0.3/skills/brainstorming/SKILL.md
```

Resolution:
```
superpowers:brainstorming
  → plugin_name = "superpowers"
  → skill_name = "brainstorming"
  → Search: $HOME/.claude/plugins/cache/**/{plugin_name}/*/skills/{skill_name}/SKILL.md
```

**Note**: When using the Glob tool, resolve `$HOME` to the actual home directory path before constructing the glob pattern. The Glob tool does not perform environment variable expansion.

**If skill has no ":" (plugin-local):**
```
security-review
  → Look in: ${CLAUDE_PLUGIN_ROOT}/skills/{skill_name}/SKILL.md
```

### 2.5. Version Selection

When multiple versions of an external plugin are found:

1. **Prefer most recently modified**: Select the SKILL.md file with the most recent modification timestamp
2. **Rationale**: The most recently modified version typically corresponds to the latest installed or updated plugin

This handles cases where a user has multiple plugin versions cached (e.g., after updates).

### 3. Handle Not Found

If skill file not found after searching all paths:

1. **Log detailed warning**:
   ```
   Warning: Skill '{plugin_name}:{skill_name}' not found.
   Searched paths:
     - $HOME/.claude/plugins/cache/**/{plugin_name}/*/skills/{skill_name}/SKILL.md
   Continuing with remaining skills.
   ```

2. **Continue processing**: Do not fail the entire command - process remaining skills

3. **Track failed skills**: Maintain list of skills that failed to resolve

4. **Report in output**: If any skills failed to resolve, include a warning in the review output:
   ```markdown
   > **Warning:** The following skills could not be resolved and were skipped:
   > - superpowers:nonexistent-skill
   ```

### 4. Extract Methodology Content

Read the SKILL.md file and extract relevant sections:

**Include:**
- Category/checklist sections (e.g., "Security Categories Checked")
- Methodology descriptions
- Pattern lists and examples
- Focus areas

**Exclude:**
- YAML frontmatter
- "When to Use" section
- "Process Overview" section (interactive workflow)
- References to other files

### 5. Format for Embedding

```yaml
embedded_skills:
  - name: "{skill_name}"
    source: "{original_skill_reference}"
    methodology: |
      {extracted content}
```

## Usage

Commands call this resolver in Step 3.5 (after Content Gathering, before Review Execution):

1. Parse `--skills` from arguments
2. For each skill, resolve and extract using this algorithm
3. Collect all embedded_skills entries
4. Pass to Step 4 for inclusion in agent prompts
