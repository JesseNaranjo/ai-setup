# Skill Loading and Interpretation

This file defines the skill loading process used by all review commands when `--skills` is provided.

## When to Apply

If `--skills` argument is present, load and interpret skill content for orchestration.

## Skill Loading Process

**CRITICAL**: The orchestrator MUST use the Skill() tool as the primary method for loading skills. Direct file read is ONLY permitted as a fallback when the Skill() tool fails.

See `${CLAUDE_PLUGIN_ROOT}/shared/skill-resolver.md` for the complete resolution algorithm.

For each skill in the comma-separated list:
1. **FIRST: Invoke Skill() tool** to load the skill (e.g., `Skill(skill: "security-review")`)
2. **ONLY IF Skill() tool fails**, fall back to direct SKILL.md file read per skill-resolver.md
3. Parse skill into structured representation (`resolved_skills`)
4. Store for orchestration decisions in the review step

### Skill-Informed Orchestration

See `${CLAUDE_PLUGIN_ROOT}/shared/review-workflow.md` "Skill-Informed Orchestration" section for how the orchestrator uses resolved skills to:
- Generate agent-specific `skill_instructions` (tailored per agent)
- Apply `auto_validated_patterns` during validation
- Apply `false_positive_rules` across agents
- Use methodology skills universally for all agent invocations
