# Using skill_instructions in Agents

When `skill_instructions` is present in the prompt, apply it as follows:

## Primary Agent Instructions

For agents that receive review-focused skill data (security-review → security-agent, bug-review → bug-detection-agent, performance-review → performance-agent, compliance-review → compliance-agent):

1. **focus_areas**: Prioritize checking these categories FIRST before standard checks. Structure findings around these areas where applicable.

2. **checklist**: For each checklist category, explicitly verify EVERY item. If an item is clean (no issues found), still acknowledge it was checked. Report: "Checked [item]: No issues found" for clean items.

3. **auto_validate**: Issues matching these pattern IDs should include `auto_validated: true` in output. These skip the validation phase.

4. **false_positive_rules**: Apply these as ADDITIONAL false positive filters beyond the standard rules in this agent.

## Methodology Instructions

From methodology skills like `superpowers:brainstorming`:

1. **methodology.approach**: Adopt this mindset throughout analysis
2. **methodology.steps**: Follow these steps as part of your review process
3. **methodology.questions**: Consider these questions when evaluating each potential finding

## Synthesis Agent Instructions

The synthesis-agent receives methodology skills only (not review-focused skills like security-review).

Methodology skills are applied the same way as for other agents:

1. **methodology.approach**: Adopt this mindset throughout cross-category analysis
2. **methodology.steps**: Follow these steps when identifying cross-cutting concerns
3. **methodology.questions**: Consider these questions when evaluating potential ripple effects between categories

**Note**: Synthesis agents do NOT receive primary agent skill data (focus_areas, checklist, auto_validate, false_positive_rules) since they operate on findings from other agents rather than reviewing code directly.

## When skill_instructions is Absent

Proceed with standard review process defined in this agent.
