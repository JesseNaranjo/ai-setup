# Skill Handling: Resolution and Orchestration

This document combines skill resolution and orchestration for `--skills` argument processing.

## Contents

- [Part 1: Skill Resolution](#part-1-skill-resolution)
  - [Skill Loading Efficiency](#skill-loading-efficiency)
  - [Resolution Algorithm](#resolution-algorithm)
  - [Structured Data Extraction](#structured-data-extraction)
  - [Parsing Rules and Steps](#parsing-rules-and-steps)
  - [Build Resolved Skills Structure](#build-resolved-skills-structure)
- [Part 2: Skill-Informed Orchestration](#part-2-skill-informed-orchestration)
  - [Agent Selection Adjustments](#agent-selection-adjustments)
  - [Agent Prompt Generation](#agent-prompt-generation)
  - [Validation Phase Adjustments](#validation-phase-adjustments)
  - [Synthesis Phase Adjustments](#synthesis-phase-adjustments)
  - [Skill Instructions in Gaps Mode](#skill-instructions-in-gaps-mode)
  - [skill_instructions Generation Algorithm](#skill_instructions-generation-algorithm)

---

# Part 1: Skill Resolution

Resolve skill names to SKILL.md files and parse them into structured data for orchestrator interpretation.

## Skill Loading Efficiency

The skill resolution process is designed for context efficiency:

1. **Metadata phase**: Only name and description are evaluated to determine applicability
2. **Content phase**: Full SKILL.md content is loaded only for applicable skills
3. **Reference phase**: Files in `references/` and `examples/` are loaded on-demand

This progressive disclosure pattern minimizes context window usage while maintaining full capability access.

## Resolution Algorithm

For each skill in the `--skills` argument:

### 1. Parse Skill Name

Split on comma to get individual skill names.

### 2. Load Skill via Skill() Tool (MANDATORY)

**CRITICAL**: The orchestrator MUST invoke the Skill() tool for EVERY skill. Direct file read is NEVER the first approach—it is ONLY a fallback when the Skill() tool fails.

For each skill name, the orchestrator MUST invoke the Skill() tool:

```
Skill(skill: "{skill_name}")
```

**Examples:**
- `reviewing-security` → `Skill(skill: "reviewing-security")`
- `superpowers:brainstorming` → `Skill(skill: "superpowers:brainstorming")`

The Skill() tool will:
1. Resolve the skill to its SKILL.md file
2. Load the skill content into context
3. Return the skill content for parsing

**On Success:** Proceed to Step 4 (Parse Skill Content) with the loaded content.

### 2.5. Fallback: Direct File Read (On Skill() Tool Failure)

If the Skill() tool returns an error or fails to load the skill:

1. **Log warning**:
   ```
   Warning: Skill() tool failed for '{skill_name}'. Falling back to direct file read.
   Error: {error_message}
   ```

2. **Attempt direct file resolution**:

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
   reviewing-security
     → Look in: ${CLAUDE_PLUGIN_ROOT}/skills/{skill_name}/SKILL.md
   ```

3. **Version selection** (when multiple external versions found):
   - Prefer most recently modified SKILL.md file
   - Rationale: Most recent version typically corresponds to latest installed plugin

4. **Read file** using Read tool if path found

5. **On fallback success**: Proceed to Step 4 (Parse Skill Content)

6. **On fallback failure**: Proceed to Step 3 (Handle Not Found)

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

## Structured Data Extraction

**For review-focused skills (reviewing-bugs, reviewing-compliance, reviewing-performance, reviewing-security, reviewing-technical-debt):**

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

## Parsing Rules and Steps

**Rules:**
1. **Strip YAML frontmatter** - Not included in parsed output
2. **Ignore "When to Use" sections** - These are for skill selection, not execution
3. **Ignore "Process Overview" sections** - Interactive workflow, not applicable to batch review
4. **Preserve category structure** - Keep hierarchy (category → checks)
5. **Extract severity from context** - If a category mentions "Critical" or "Major", capture it

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
- If name matches `reviewing-architecture-principles|reviewing-bugs|reviewing-compliance|reviewing-performance|reviewing-security|reviewing-technical-debt` → `type: "review"`
- If name matches `reviewing-documentation` → `type: "command"` (meta-skill that invokes docs-review commands)
- Otherwise → `type: "methodology"`

**Step 4.6: Assign Primary Agent**

| Skill Name | Primary Agent |
|------------|---------------|
| reviewing-architecture-principles | architecture-agent |
| reviewing-bugs | bug-detection-agent |
| reviewing-compliance | compliance-agent |
| reviewing-documentation | N/A (command-invoking meta-skill) |
| reviewing-performance | performance-agent |
| reviewing-security | security-agent |
| reviewing-technical-debt | technical-debt-agent |
| (methodology) | null |

**Meta-Skills:** `reviewing-documentation` is a meta-skill that invokes documentation review COMMANDS (`/deep-docs-review`, `/quick-docs-review`) rather than targeting individual agents. It should be invoked directly, not passed as `--skills` to other commands.

## Build Resolved Skills Structure

```yaml
resolved_skills:
  - name: "reviewing-security"
    source: "reviewing-security"  # Original reference
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

---

# Part 2: Skill-Informed Orchestration

When `--skills` is provided, the orchestrator interprets the resolved skills and makes orchestration decisions. This section defines how skill data affects each phase of the review.

## Agent Selection Adjustments

Skills can affect which agents run and with what configuration:

**Override skip_agents:**
- If a skill has `primary_agent: "security-agent"` and security-agent is in `skip_agents`, the orchestrator may warn but does not auto-override
- Explicit user configuration takes precedence

**Model Selection:**
- Skills do not change the model selection per agent
- The authoritative model table in `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md` always applies

## Agent Prompt Generation

Instead of passing raw skill content to agents via `embedded_skills`, the orchestrator generates tailored `skill_instructions` for each agent.

**Template for skill_instructions:**

```yaml
skill_instructions:
  # For review-focused skills targeting this agent
  focus_areas:
    - "OWASP Top 10 vulnerabilities"
    - "Injection attacks (SQL, command, XSS)"
  checklist:
    - category: "Injection Vulnerabilities"
      severity: "Critical"
      items:
        - "SQL injection via string concatenation"
        - "Command injection via exec/spawn"
    - category: "Authentication Weaknesses"
      severity: "Critical"
      items:
        - "Hardcoded credentials"
        - "Weak token generation"
  auto_validate:
    - "hardcoded_password"
    - "sql_concat"
  false_positive_rules:
    - "Passwords in test files or fixtures"
    - "SQL in ORM query builders"

  # For methodology skills (applies to all agents)
  methodology:
    approach: "Explore multiple interpretations before concluding"
    mindset: "Question assumptions, consider edge cases"
    steps:
      - "Identify what the code is trying to do"
      - "Consider why it might be correct"
      - "Brainstorm ways it could fail"
    questions:
      - "What assumptions does this code make?"
      - "What happens if those assumptions are violated?"
```

**Generation Rules:**

| Agent | Receives From |
|-------|---------------|
| architecture-agent | `reviewing-architecture-principles` focus_areas, ALL methodology skills |
| bug-detection-agent | `reviewing-bugs` focus_areas, ALL methodology skills |
| compliance-agent | `reviewing-compliance` focus_areas, ALL methodology skills |
| performance-agent | `reviewing-performance` focus_areas, ALL methodology skills |
| security-agent | `reviewing-security` focus_areas, ALL methodology skills |
| technical-debt-agent | `reviewing-technical-debt` focus_areas, ALL methodology skills |
| Other agents | ALL methodology skills only |

**Merging Multiple Skills:**

When multiple skills are provided (e.g., `--skills reviewing-security,superpowers:brainstorming`):

1. **focus_areas**: Union of all applicable focus areas
2. **checklist**: Union of all applicable checklists
3. **auto_validate**: Union of all patterns
4. **false_positive_rules**: Union of all rules
5. **methodology**: Include all methodology skills in sequence

## Validation Phase Adjustments

Skills affect validation through:

**Auto-Validated Patterns:**
- Issues matching `auto_validated_patterns` skip the validation subagent
- They are automatically marked as VALID
- Rationale: These patterns are high-confidence findings that don't need secondary verification

**False Positive Rules:**
- Pass `false_positive_rules` to the validation prompt
- Validator uses these as additional context for INVALID verdicts

## Synthesis Phase Adjustments

Skills can inform synthesis questions:

**Default Cross-Cutting Questions:**
- Use the standard questions defined in `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md`

**With Skills:**
- If `reviewing-security` skill is active, add: "Do findings align with OWASP categories?"
- If `reviewing-performance` skill is active, add: "Are performance issues in hot paths?"
- Methodology skills don't affect synthesis questions

## Skill Instructions in Gaps Mode

When `--skills` is provided, gaps mode agents receive the **same** `skill_instructions` as thorough mode agents.

**Rationale:**
- Gaps mode agents look for subtle issues that thorough mode missed
- Skill-defined focus areas and checklists help gaps agents find category-specific edge cases
- Methodology skills (e.g., `superpowers:brainstorming`) apply equally to gaps analysis

**Skill Application to Gaps Mode:**

| Skill Element | Applied to Gaps Mode? | Notes |
|---------------|----------------------|-------|
| `focus_areas` | Yes | Same focus areas help find subtle issues |
| `checklist` | Yes | Gaps mode checks same items, looking for edge cases |
| `auto_validated_patterns` | Yes | Auto-validated patterns skip validation regardless of mode |
| `false_positive_rules` | Yes | Applied during gaps mode validation |
| `methodology` | Yes | Methodology applies to all agent thinking |

**Implementation:**
- Gaps mode agents receive identical `skill_instructions` block as thorough mode
- Validation phase applies skill-derived `auto_validated_patterns` to ALL findings (thorough + gaps)
- Validation phase applies skill-derived `false_positive_rules` to ALL findings (thorough + gaps)

## skill_instructions Generation Algorithm

When `--skills` is provided, generate `skill_instructions` for each agent BEFORE launching agents in Step 4.

**Generation Step 1: Initialize per-agent containers**

Create empty skill_instructions structures for each of the 10 agents:
```yaml
agent_skill_instructions:
  security-agent: { focus_areas: [], checklist: [], auto_validate: [], false_positive_rules: [], methodology: null }
  bug-detection-agent: { focus_areas: [], checklist: [], auto_validate: [], false_positive_rules: [], methodology: null }
  performance-agent: { focus_areas: [], checklist: [], auto_validate: [], false_positive_rules: [], methodology: null }
  compliance-agent: { focus_areas: [], checklist: [], auto_validate: [], false_positive_rules: [], methodology: null }
  architecture-agent: { focus_areas: [], checklist: [], auto_validate: [], false_positive_rules: [], methodology: null }
  api-contracts-agent: { focus_areas: [], checklist: [], auto_validate: [], false_positive_rules: [], methodology: null }
  error-handling-agent: { focus_areas: [], checklist: [], auto_validate: [], false_positive_rules: [], methodology: null }
  test-coverage-agent: { focus_areas: [], checklist: [], auto_validate: [], false_positive_rules: [], methodology: null }
  technical-debt-agent: { focus_areas: [], checklist: [], auto_validate: [], false_positive_rules: [], methodology: null }
  synthesis-agent: { focus_areas: [], checklist: [], auto_validate: [], false_positive_rules: [], methodology: null }
```

**Generation Step 2: Process each resolved skill**

For each skill in `resolved_skills`:

**If skill.type == "review":**
1. Identify target agent from `skill.primary_agent`
2. Append all items from `skill.focus_areas` to `agent_skill_instructions[target_agent].focus_areas`
3. Build checklist entries from focus_areas:
   ```yaml
   checklist:
     - category: [focus_area.name]
       severity: [focus_area.severity]
       items: [focus_area.checks]
   ```
4. Append `skill.auto_validated_patterns[].id` to `agent_skill_instructions[target_agent].auto_validate`
5. Append `skill.false_positive_rules` to `agent_skill_instructions[target_agent].false_positive_rules`

**If skill.type == "methodology":**
1. For EACH agent (all 10):
   - Set `agent_skill_instructions[agent].methodology` to `skill.methodology`
   - If methodology already exists, append to existing steps and questions

**Generation Step 3: Build final skill_instructions block**

When constructing the Task prompt for each agent, include the `skill_instructions:` block only if any field is non-empty:

```yaml
skill_instructions:
  focus_areas: [from agent_skill_instructions[agent].focus_areas]
  checklist: [from agent_skill_instructions[agent].checklist]
  auto_validate: [from agent_skill_instructions[agent].auto_validate]
  false_positive_rules: [from agent_skill_instructions[agent].false_positive_rules]
  methodology: [from agent_skill_instructions[agent].methodology, if not null]
```

Omit the entire `skill_instructions:` block if all fields are empty (no skills provided or no applicable data for this agent).

**Generation Step 4: Store patterns for validation phase**

Before launching agents, store combined auto-validation data for use in Step 5 (Validation):
```yaml
skill_validation_context:
  auto_validate_patterns: [union of all auto_validate from all skills]
  false_positive_rules: [union of all false_positive_rules from all skills]
```

Pass this context to the validation step to enable skill-derived auto-validation.
