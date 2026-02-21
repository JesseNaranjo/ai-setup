# Skill Handling

## Skill Resolution

1. Parse comma-separated `--skills`
2. Load via `Skill(skill: "{name}")` (MANDATORY). Fallback: external (`plugin:skill`) → search `$HOME/.claude/plugins/cache/**/{plugin}/*/skills/{skill}/SKILL.md` (resolve `$HOME` first; multiple versions: most recent); local → `${CLAUDE_PLUGIN_ROOT}/skills/{skill}/SKILL.md`
3. Not found: warn, continue, report skipped
4. Parse into structured data

## Structured Data Extraction

**Review skills** (reviewing-{architecture-principles,bugs,compliance,performance,security,technical-debt}): `focus_areas` ← "Additional Focus Areas" (empty if absent), `auto_validated_patterns` ← pattern tables, `false_positive_rules` ← "False Positives", `priority_files` ← "Scope Prioritization", `primary_agent` ← name mapping (below).

**Methodology skills** (superpowers:*, etc.): `methodology` ← main content, `steps` ← numbered lists/workflows, `questions` ← question-based sections.

**Parsing:** Ignore "When to Use"/"Process Overview". Preserve category hierarchy. Type: `reviewing-{architecture-principles|bugs|compliance|performance|security|technical-debt}` → review; `reviewing-documentation` → command (meta-skill); else → methodology.

## Primary Agent Mapping

Convention: `reviewing-{X}` → `{X}-agent`. Exceptions: architecture-principles→architecture, bugs→bug-detection. Special: reviewing-documentation→N/A (command meta-skill), methodology skills→null.

Command meta-skill: no primary agent. False positive rules and auto-validated patterns apply globally (added to validation context). Focus areas and priority files are not distributed to agents.

## Resolved Structure

```yaml
resolved_skills:
  - name: "reviewing-security"
    type: "review"
    primary_agent: "security-agent"
    focus_areas: [{ name, severity, checks }]
    auto_validated_patterns: [{ id, pattern, description }]
    false_positive_rules: [...]
    priority_files: { patterns: [...] }
  - { name: "brainstorming", source: "superpowers:brainstorming", type: "methodology", methodology: { approach, mindset, steps, questions } }
```

## Skill-Informed Orchestration

### Agent Instruction Distribution

Primary agents (architecture, bug-detection, compliance, performance, security, technical-debt): matching review skill focus_areas + ALL methodology. Other agents (api-contracts, error-handling, synthesis-*, test-coverage): methodology only.

**Multiple skills:** Union all focus_areas, checklists, auto_validate patterns, false_positive_rules. All methodology in sequence.

### skill_instructions Generation

Per agent, build: focus_areas, checklist, auto_validate, false_positive_rules, methodology.

**Review skill:** focus_areas non-empty → append to primary agent + build checklist (category, severity, items); empty → skip. Always: append auto_validated_patterns[].id to auto_validate; append false_positive_rules.
**Methodology skill:** All agents get methodology (steps, questions).

Include only if non-empty. Store for validation: `skill_validation_context: { auto_validate_patterns: [union], false_positive_rules: [union] }`

### Validation Adjustments

- Auto-validated patterns: skip validation subagent, auto-mark VALID
- False positive rules: passed as additional INVALID context

### Synthesis Adjustments

Standard questions from orchestration file plus: reviewing-security → "Do findings align with OWASP categories?"; reviewing-performance → "Are performance issues in hot paths?"

### Agent Guidance

Inject into `additional_instructions` when `--skills` active:

1. **focus_areas**: Prioritize FIRST before standard checks; empty → standard categories
2. **checklist**: Verify EVERY item; acknowledge clean items
3. **auto_validate**: Matching issues include `auto_validated: true`
4. **false_positive_rules**: Additional FP filters beyond standard
5. **methodology**: Adopt mindset, follow steps, consider questions

No primary skill → methodology only. No skill_instructions → standard process. `skip_agents` takes precedence over skill primary_agent (warn). Skills don't change model selection. Gaps/quick receive identical skill_instructions; validation applies to ALL findings.
