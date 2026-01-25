# Skill Resolver

Resolve skill names to SKILL.md files and parse them into structured data for orchestrator interpretation.

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

### 4. Parse Skill Content

Read the SKILL.md file and parse it into structured data. The orchestrator uses this structured data to generate agent-specific instructions.

#### Structured Data Extraction

**For review-focused skills (security-review, performance-review, bug-review, compliance-review):**

| Field | Source | Description |
|-------|--------|-------------|
| `focus_areas` | "Categories Checked" sections | List of focus areas with severity and specific checks |
| `auto_validated_patterns` | Pattern tables | Patterns that auto-validate (skip validation step) |
| `false_positive_rules` | "False Positives" sections | Rules for what NOT to flag |
| `priority_files` | "Scope Prioritization" sections | File patterns to prioritize |
| `primary_agent` | Skill name mapping | Which agent this skill primarily targets |

**For methodology-focused skills (superpowers:brainstorming, superpowers:systematic-debugging, etc.):**

| Field | Source | Description |
|-------|--------|-------------|
| `methodology` | Main content | Approach/mindset description |
| `steps` | Numbered lists or workflow sections | Ordered steps to follow |
| `questions` | Question-based sections | Exploratory questions to ask |

#### Parsing Rules

1. **Strip YAML frontmatter** - Not included in parsed output
2. **Ignore "When to Use" sections** - These are for skill selection, not execution
3. **Ignore "Process Overview" sections** - Interactive workflow, not applicable to batch review
4. **Preserve category structure** - Keep hierarchy (category → checks)
5. **Extract severity from context** - If a category mentions "Critical" or "Major", capture it

#### Parsing Execution Steps

For each resolved SKILL.md file, execute these steps to extract structured data:

**Step 4.1: Read and Split**
1. Read the SKILL.md file content using the Read tool
2. Identify YAML frontmatter (content between first two `---` markers)
3. Extract body content (everything after the closing `---`)

**Step 4.2: Extract Focus Areas**
Search the body for sections with headers containing severity indicators:

```
Pattern: ## [Section Name] (Critical|Major|Minor)
Pattern: ### [Section Name] (Critical|Major|Minor):
```

For each matching section:
1. Extract section name before the parentheses as `focus_area.name`
2. Extract severity from parentheses as `focus_area.severity`
3. Parse bullet points (lines starting with `-`) under the section as `focus_area.checks[]`

**Example:**
Section "### Injection Vulnerabilities (Critical):" with bullets becomes:
```yaml
focus_areas:
  - name: "Injection Vulnerabilities"
    severity: "Critical"
    checks:
      - "SQL injection via string concatenation"
      - "Command injection through shell execution"
```

**Step 4.3: Extract Auto-Validated Patterns**
Search for a section header containing "Auto-Validated" and a markdown table:

1. Find the table with columns containing "Pattern" and "Description"
2. For each data row (skip header and separator):
   - Column 1 (inside backticks) → `id`
   - Column 2 → `description`

**Step 4.4: Extract False Positive Rules**
Search for a section header containing "False Positive":

1. Find all bullet points in that section
2. Each bullet (text after `- `) becomes an entry in `false_positive_rules[]`

**Step 4.5: Determine Skill Type**
Based on skill name:
- If name matches `security-review|performance-review|bug-review|compliance-review` → `type: "review"`
- Otherwise → `type: "methodology"`

**Step 4.6: Assign Primary Agent**

| Skill Name | Primary Agent |
|------------|---------------|
| security-review | security-agent |
| performance-review | performance-agent |
| bug-review | bug-detection-agent |
| compliance-review | compliance-agent |
| (methodology) | null |

### 5. Build Resolved Skills Structure

```yaml
resolved_skills:
  - name: "security-review"
    source: "security-review"  # Original reference
    type: "review"  # review | methodology
    primary_agent: "security-agent"  # null for methodology skills
    focus_areas:
      - name: "Injection Vulnerabilities"
        severity: "Critical"
        checks:
          - "SQL injection via string concatenation"
          - "Command injection via exec/spawn"
          - "XSS via innerHTML or dangerouslySetInnerHTML"
      - name: "Authentication Weaknesses"
        severity: "Critical"
        checks:
          - "Hardcoded credentials"
          - "Weak token generation"
    auto_validated_patterns:
      - id: "hardcoded_password"
        pattern: "password\\s*=\\s*[\"'][^\"']+[\"']"
        description: "Hardcoded password assignments"
      - id: "sql_concat"
        pattern: "\\+.*\\$\\{.*\\}.*FROM|WHERE"
        description: "SQL string concatenation with variables"
    false_positive_rules:
      - "Passwords in test files or fixtures"
      - "SQL in ORM query builders"
      - "Example code in documentation"
    priority_files:
      patterns:
        - "**/auth/**"
        - "**/login/**"
        - "**/password/**"
        - "**/api/**"

  - name: "brainstorming"
    source: "superpowers:brainstorming"
    type: "methodology"
    primary_agent: null  # Applies to all agents
    methodology:
      approach: "Explore multiple interpretations before concluding"
      mindset: "Question assumptions, consider edge cases"
      steps:
        - "Identify what the code is trying to do"
        - "Consider why it might be correct"
        - "Brainstorm ways it could fail"
        - "Evaluate likelihood of each failure mode"
      questions:
        - "What assumptions does this code make?"
        - "What happens if those assumptions are violated?"
        - "Are there edge cases the developer might have missed?"
```

## Usage

Commands call this resolver in Step 5 (Skill Loading and Interpretation):

1. Parse `--skills` from arguments
2. For each skill, resolve file path and read content
3. Parse content into structured `resolved_skills` entries
4. Store for orchestration decisions in subsequent steps

The orchestrator (command running as Opus) then:
- Uses `primary_agent` to route skill-specific checks
- Generates tailored `skill_instructions` per agent
- Applies `auto_validated_patterns` during validation phase
- Applies `false_positive_rules` across all agents
- Uses `methodology` skills universally for all agent invocations
