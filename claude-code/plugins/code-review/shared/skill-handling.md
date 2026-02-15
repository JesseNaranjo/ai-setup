# Skill Handling: Resolution and Orchestration

# Part 1: Skill Resolution

Resolve skill names from `--skills` argument to SKILL.md files and parse into structured data.

## Resolution Algorithm

### 1. Parse Skill Name
Split on comma to get individual skill names.

### 2. Load Skill via Skill() Tool (MANDATORY)

**CRITICAL**: Invoke `Skill(skill: "{skill_name}")` for EVERY skill. Direct file read is ONLY a fallback when Skill() fails.

On success, proceed to Step 4.

### 2.5. Fallback: Direct File Read (On Skill() Tool Failure)

**External plugin (contains ":"):** Parse `superpowers:brainstorming` → `plugin_name = "superpowers"`, `skill_name = "brainstorming"`. Search: `$HOME/.claude/plugins/cache/**/{plugin_name}/*/skills/{skill_name}/SKILL.md`. Resolve `$HOME` to actual path before globbing (Glob does not expand env vars). Multiple versions: prefer most recently modified.

**Plugin-local (no ":"):** `${CLAUDE_PLUGIN_ROOT}/skills/{skill_name}/SKILL.md`

### 3. Handle Not Found
Log warning, continue processing remaining skills, report skipped skills in output.

### 4. Parse Skill Content
Parse SKILL.md into structured data for agent-specific instruction generation.

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

1. **Ignore** "When to Use" and "Process Overview" sections (skill selection/interactive workflow, not batch review)
2. **Preserve** category hierarchy (category → checks) and extract severity from context

**Step 4.5: Determine Skill Type:**
- `reviewing-architecture-principles|reviewing-bugs|reviewing-compliance|reviewing-performance|reviewing-security|reviewing-technical-debt` → `type: "review"`
- `reviewing-documentation` → `type: "command"` (meta-skill invoking docs-review commands)
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

**Meta-Skills:** `reviewing-documentation` invokes docs review commands directly, not via `--skills` on other commands.

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

- `skip_agents` takes precedence over skill `primary_agent` (warn but don't auto-override)
- Skills do not change model selection — authoritative model table in `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md` always applies

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

- **Auto-Validated Patterns:** Issues matching `auto_validated_patterns` skip validation subagent, auto-marked VALID
- **False Positive Rules:** Passed to validation prompt as additional INVALID context

## Synthesis Phase Adjustments

Standard questions from `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md` plus skill-specific additions:
- `reviewing-security`: "Do findings align with OWASP categories?"
- `reviewing-performance`: "Are performance issues in hot paths?"

## Skill Instructions in Gaps Mode

Gaps/quick agents receive identical `skill_instructions` as thorough. Validation applies skill-derived patterns/rules to ALL findings.

## skill_instructions Generation Algorithm

When `--skills` is provided, generate `skill_instructions` for each agent BEFORE launching agents in Step 4.

**Step 1:** Initialize empty `skill_instructions` containers (`focus_areas`, `checklist`, `auto_validate`, `false_positive_rules`, `methodology`) per agent.

**Step 2:** For each skill in `resolved_skills`:

**If skill.type == "review":**
1. Identify target agent from `skill.primary_agent`
2. Append `skill.focus_areas` to the target agent's `focus_areas`
3. Build checklist entries from focus_areas (category name, severity, checks as items)
4. Append `skill.auto_validated_patterns[].id` to the target agent's `auto_validate`
5. Append `skill.false_positive_rules` to the target agent's `false_positive_rules`

**If skill.type == "methodology":**
1. For EACH agent: set `methodology` from `skill.methodology` (if already exists, append to existing steps and questions)

**Step 3:** Include `skill_instructions` in agent prompt only if any field is non-empty.

**Step 4:** Store combined auto-validation data for validation phase:
```yaml
skill_validation_context:
  auto_validate_patterns: [union of all auto_validate from all skills]
  false_positive_rules: [union of all false_positive_rules from all skills]
```

Pass this context to the validation step.

## Skill Instructions Agent Guidance

Inject into each agent's `additional_instructions` when `--skills` is active:

When `skill_instructions` is present:
1. **focus_areas**: Prioritize these categories FIRST before standard checks
2. **checklist**: Explicitly verify EVERY item; acknowledge clean items
3. **auto_validate**: Matching issues include `auto_validated: true` in output
4. **false_positive_rules**: Apply as ADDITIONAL FP filters beyond standard rules
5. **methodology.approach/steps/questions**: Adopt mindset, follow steps, consider questions throughout analysis

Agents without a primary review skill (api-contracts, error-handling, synthesis-*, test-coverage) receive only methodology. When `skill_instructions` is absent, use standard review process.
