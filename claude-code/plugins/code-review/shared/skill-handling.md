# Skill Handling: Resolution and Orchestration

This document combines skill resolution and orchestration for `--skills` argument processing.

# Part 1: Skill Resolution

Resolve skill names to SKILL.md files and parse them into structured data for orchestrator interpretation.

## Resolution Algorithm

For each skill in the `--skills` argument:

### 1. Parse Skill Name
Split on comma to get individual skill names.

### 2. Load Skill via Skill() Tool (MANDATORY)

**CRITICAL**: The orchestrator MUST invoke the Skill() tool for EVERY skill. Direct file read is NEVER the first approach—it is ONLY a fallback when the Skill() tool fails.

```
Skill(skill: "{skill_name}")
```

On success, proceed to Step 4 (Parse Skill Content).

### 2.5. Fallback: Direct File Read (On Skill() Tool Failure)
If the Skill() tool returns an error or fails to load the skill, attempt direct file resolution:

**If skill contains ":" (external plugin):**
Claude native install path structure (where `$HOME` is the user's home directory):
```
$HOME/.claude/plugins/cache/{source}/{plugin_name}/{version}/skills/{skill_name}/SKILL.md
```

Parse the skill reference: `superpowers:brainstorming` → `plugin_name = "superpowers"`, `skill_name = "brainstorming"`. Search: `$HOME/.claude/plugins/cache/**/{plugin_name}/*/skills/{skill_name}/SKILL.md`. **Note**: Resolve `$HOME` to the actual path before constructing the glob pattern (Glob does not expand environment variables).

**If skill has no ":" (plugin-local):**
```
${CLAUDE_PLUGIN_ROOT}/skills/{skill_name}/SKILL.md
```

When multiple external versions are found, prefer the most recently modified SKILL.md file. On fallback success, proceed to Step 4. On fallback failure, proceed to Step 3.

### 3. Handle Not Found
If skill file not found after searching all paths: log a warning, continue processing remaining skills, track failed skills, and report skipped skills in the review output.

### 4. Parse Skill Content
Read the SKILL.md file and parse it into structured data. The orchestrator uses this structured data to generate agent-specific instructions.

## Structured Data Extraction

**Review-focused skills (reviewing-bugs, reviewing-compliance, reviewing-performance, reviewing-security, reviewing-technical-debt):**

| Field | Source | Description |
|-------|--------|-------------|
| `focus_areas` | "Categories Checked" sections | List of focus areas with severity and specific checks |
| `auto_validated_patterns` | Pattern tables | Patterns that auto-validate (skip validation step) |
| `false_positive_rules` | "False Positives" sections | Rules for what NOT to flag |
| `priority_files` | "Scope Prioritization" sections | File patterns to prioritize |
| `primary_agent` | Skill name mapping | Which agent this skill primarily targets |

**Methodology-focused skills (superpowers:brainstorming, superpowers:systematic-debugging, etc.):**

| Field | Source | Description |
|-------|--------|-------------|
| `methodology` | Main content | Approach/mindset description |
| `steps` | Numbered lists or workflow sections | Ordered steps to follow |
| `questions` | Question-based sections | Exploratory questions to ask |

## Parsing Rules

1. **Ignore "When to Use" sections** - These are for skill selection, not execution
2. **Ignore "Process Overview" sections** - Interactive workflow, not applicable to batch review
3. **Preserve category structure** - Keep hierarchy (category → checks)
4. **Extract severity from context** - If a category mentions "Critical" or "Major", capture it

**Step 4.5: Determine Skill Type** - Based on skill name:
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

**Meta-Skills:** `reviewing-documentation` invokes documentation review commands (`/deep-docs-review`, `/quick-docs-review`) rather than targeting individual agents. Invoke directly, not via `--skills` on other commands.

## Build Resolved Skills Structure

```yaml
resolved_skills:
  - name: "reviewing-security"
    source: "reviewing-security"
    type: "review"
    primary_agent: "security-agent"
    focus_areas: [{ name: "...", severity: "Critical", checks: [...] }]
    auto_validated_patterns: [{ id: "...", pattern: "...", description: "..." }]
    false_positive_rules: ["..."]
    priority_files: { patterns: ["**/auth/**", ...] }
  - name: "brainstorming"
    source: "superpowers:brainstorming"
    type: "methodology"
    primary_agent: null
    methodology: { approach: "...", mindset: "...", steps: [...], questions: [...] }
```

---

# Part 2: Skill-Informed Orchestration

When `--skills` is provided, the orchestrator interprets the resolved skills and makes orchestration decisions.

## Agent Selection Adjustments

**Override skip_agents:**
- If a skill has `primary_agent: "security-agent"` and security-agent is in `skip_agents`, the orchestrator may warn but does not auto-override
- Explicit user configuration takes precedence

**Model Selection:** Skills do not change the model selection per agent. The authoritative model table in `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md` always applies.

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

**Merging Multiple Skills** (e.g., `--skills reviewing-security,superpowers:brainstorming`):

1. **focus_areas**: Union of all applicable focus areas
2. **checklist**: Union of all applicable checklists
3. **auto_validate**: Union of all patterns
4. **false_positive_rules**: Union of all rules
5. **methodology**: Include all methodology skills in sequence

## Validation Phase Adjustments

**Auto-Validated Patterns:** Issues matching `auto_validated_patterns` skip the validation subagent and are automatically marked as VALID (high-confidence findings that don't need secondary verification).

**False Positive Rules:** Pass `false_positive_rules` to the validation prompt. Validator uses these as additional context for INVALID verdicts.

## Synthesis Phase Adjustments

Use the standard questions from `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md`. With skills active, add skill-specific questions:
- `reviewing-security`: "Do findings align with OWASP categories?"
- `reviewing-performance`: "Are performance issues in hot paths?"
- Methodology skills don't affect synthesis questions

## Skill Instructions in Gaps Mode

Gaps and quick mode agents receive identical `skill_instructions` as thorough mode. Validation applies skill-derived `auto_validated_patterns` and `false_positive_rules` to ALL findings (thorough + gaps + quick).

## skill_instructions Generation Algorithm

When `--skills` is provided, generate `skill_instructions` for each agent BEFORE launching agents in Step 4.

**Generation Step 1: Initialize per-agent containers**
Initialize empty `skill_instructions` containers (`focus_areas`, `checklist`, `auto_validate`, `false_positive_rules`, `methodology`) for each agent.

**Generation Step 2: Process each resolved skill**
For each skill in `resolved_skills`:

**If skill.type == "review":**
1. Identify target agent from `skill.primary_agent`
2. Append `skill.focus_areas` to the target agent's `focus_areas`
3. Build checklist entries from focus_areas (category name, severity, checks as items)
4. Append `skill.auto_validated_patterns[].id` to the target agent's `auto_validate`
5. Append `skill.false_positive_rules` to the target agent's `false_positive_rules`

**If skill.type == "methodology":**
1. For EACH agent: set `methodology` from `skill.methodology` (if already exists, append to existing steps and questions)

**Generation Step 3: Build final skill_instructions block**
Include `skill_instructions` block in agent Task prompt only if any field is non-empty. Omit entirely if no applicable skill data for that agent.

**Generation Step 4: Store patterns for validation phase**
Before launching agents, store combined auto-validation data for use in Step 5 (Validation):
```yaml
skill_validation_context:
  auto_validate_patterns: [union of all auto_validate from all skills]
  false_positive_rules: [union of all false_positive_rules from all skills]
```

Pass this context to the validation step to enable skill-derived auto-validation.

## Skill Instructions Agent Guidance

When `--skills` is active, inject the following into each agent's `additional_instructions`:

When `skill_instructions` is present in the prompt, apply it as follows:

1. **focus_areas**: Prioritize checking these categories FIRST before standard checks. Structure findings around these areas where applicable.
2. **checklist**: For each checklist category, explicitly verify EVERY item. If an item is clean (no issues found), acknowledge it was checked.
3. **auto_validate**: Issues matching these pattern IDs should include `auto_validated: true` in output.
4. **false_positive_rules**: Apply these as ADDITIONAL false positive filters beyond this agent's standard rules.

For methodology skills (like `superpowers:brainstorming`):
1. **methodology.approach**: Adopt this mindset throughout analysis
2. **methodology.steps**: Follow these steps as part of your review process
3. **methodology.questions**: Consider these questions when evaluating each potential finding

**Note:** Agents without a primary review skill (api-contracts-agent, error-handling-agent, synthesis-code-agent, synthesis-docs-agent, test-coverage-agent) receive only the methodology section.

When `skill_instructions` is absent, proceed with standard review process.
