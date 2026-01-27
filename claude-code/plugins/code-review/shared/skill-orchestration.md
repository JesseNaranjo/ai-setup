# Skill-Informed Orchestration

When `--skills` is provided, the orchestrator interprets the resolved skills and makes orchestration decisions. This document defines how skill data affects each phase of the review.

## Skill Loading

**CRITICAL**: See `${CLAUDE_PLUGIN_ROOT}/shared/skill-resolver.md` for the mandatory skill loading protocol (Skill() tool first, file read fallback only on failure).

After loading, extract structured data from each skill into `resolved_skills` for orchestration decisions.

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
| architecture-agent | `architecture-principles-review` focus_areas, ALL methodology skills |
| bug-detection-agent | `bug-review` focus_areas, ALL methodology skills |
| compliance-agent | `compliance-review` focus_areas, ALL methodology skills |
| performance-agent | `performance-review` focus_areas, ALL methodology skills |
| security-agent | `security-review` focus_areas, ALL methodology skills |
| technical-debt-agent | `technical-debt-review` focus_areas, ALL methodology skills |
| Other agents | ALL methodology skills only |

**Merging Multiple Skills:**

When multiple skills are provided (e.g., `--skills security-review,superpowers:brainstorming`):

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
- Use the standard questions defined in review-workflow.md

**With Skills:**
- If `security-review` skill is active, add: "Do findings align with OWASP categories?"
- If `performance-review` skill is active, add: "Are performance issues in hot paths?"
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
