# Skill Resolver

Resolve skill names to SKILL.md files and extract methodology content for embedding in agent prompts.

## Resolution Algorithm

For each skill in the `--skills` argument:

### 1. Parse Skill Name

Split on comma to get individual skill names.

### 2. Resolve to File Path

**If skill contains ":" (external plugin):**
```
superpowers:brainstorming
  → plugin_name = "superpowers"
  → skill_name = "brainstorming"
  → Search: ~/.claude/plugins/cache/**/{plugin_name}/*/skills/{skill_name}/SKILL.md
  → Fallback: ~/.claude/plugins/**/{plugin_name}/*/skills/{skill_name}/SKILL.md
```

**If skill has no ":" (plugin-local):**
```
security-review
  → Look in: ${CLAUDE_PLUGIN_ROOT}/skills/{skill_name}/SKILL.md
```

### 3. Handle Not Found

If skill file not found:
- Log warning: "Skill '{skill_name}' not found, skipping"
- Continue with remaining skills

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
