# Skill Handling

## Skill Resolution

Resolve `--skills` argument to SKILL.md files.

1. Parse comma-separated skill names
2. Load via `Skill(skill: "{name}")` (MANDATORY). Fallback on failure:
   - External (contains `:`): Parse `plugin:skill` → search `$HOME/.claude/plugins/cache/**/{plugin}/*/skills/{skill}/SKILL.md`. Resolve `$HOME` first (Glob doesn't expand env vars). Multiple versions: prefer most recent.
   - Local (no `:`): `${CLAUDE_PLUGIN_ROOT}/skills/{skill}/SKILL.md`
3. Not found: warn, continue, report skipped
4. Parse into structured data

## Structured Data Extraction

**Review skills** (reviewing-{architecture-principles,bugs,compliance,performance,security,technical-debt}):

| Field | Source |
|-------|--------|
| `focus_areas` | "Additional Focus Areas" sections (if present, otherwise empty) |
| `auto_validated_patterns` | Pattern tables |
| `false_positive_rules` | "False Positives" sections |
| `priority_files` | "Scope Prioritization" sections |
| `primary_agent` | Skill name mapping (see table below) |

**Methodology skills** (superpowers:*, etc.):

| Field | Source |
|-------|--------|
| `methodology` | Main content |
| `steps` | Numbered lists/workflow sections |
| `questions` | Question-based sections |

**Parsing:** Ignore "When to Use"/"Process Overview". Preserve category hierarchy. Type: `reviewing-{architecture-principles|bugs|compliance|performance|security|technical-debt}` → review; `reviewing-documentation` → command (meta-skill); else → methodology.

## Primary Agent Mapping

| Skill | Agent |
|-------|-------|
| reviewing-architecture-principles | architecture-agent |
| reviewing-bugs | bug-detection-agent |
| reviewing-compliance | compliance-agent |
| reviewing-documentation | N/A (command meta-skill) |
| reviewing-performance | performance-agent |
| reviewing-security | security-agent |
| reviewing-technical-debt | technical-debt-agent |
| (methodology) | null |

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

| Agent | Receives |
|-------|----------|
| Primary agents (architecture, bug-detection, compliance, performance, security, technical-debt) | Matching review skill focus_areas + ALL methodology |
| Other agents (api-contracts, error-handling, synthesis-*, test-coverage) | Methodology only |

**Merging multiple skills:** Union all focus_areas, checklists, auto_validate patterns, false_positive_rules. Include all methodology in sequence.

### skill_instructions Generation

Per agent, initialize: focus_areas, checklist, auto_validate, false_positive_rules, methodology.

- **Review skill:** If focus_areas non-empty: append focus_areas to primary agent; build checklist from focus_areas (category, severity, checks as items). If focus_areas empty: skip focus_areas and checklist (agent uses standard categories). Always: append auto_validated_patterns[].id to auto_validate; append false_positive_rules
- **Methodology skill:** For ALL agents, set/append methodology (steps, questions)

Include in agent prompt only if non-empty. Store combined data for validation:

```yaml
skill_validation_context: { auto_validate_patterns: [union], false_positive_rules: [union] }
```

### Validation Adjustments

- Auto-validated patterns: skip validation subagent, auto-mark VALID
- False positive rules: passed as additional INVALID context

### Synthesis Adjustments

Standard questions from orchestration file plus: reviewing-security → "Do findings align with OWASP categories?"; reviewing-performance → "Are performance issues in hot paths?"

### Agent Guidance

Inject into `additional_instructions` when `--skills` active:

1. **focus_areas**: If provided, prioritize FIRST before standard checks. If empty, use standard agent categories
2. **checklist**: Verify EVERY item; acknowledge clean items
3. **auto_validate**: Matching issues include `auto_validated: true`
4. **false_positive_rules**: Additional FP filters beyond standard
5. **methodology**: Adopt mindset, follow steps, consider questions

Agents without primary skill receive methodology only. Without skill_instructions, standard process.

## Notes

- `skip_agents` precedence over skill primary_agent (warn but don't auto-override)
- Skills don't change model selection (authoritative table in orchestration file)
- Gaps/quick agents receive identical skill_instructions; validation applies to ALL findings
