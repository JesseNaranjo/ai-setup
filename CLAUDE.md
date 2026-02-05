# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Claude Code plugin repository containing the **Code Review Plugin** (v3.4.1) - a modular 16-agent architecture with:
- Two-phase sequential review (thorough → gaps with context passing)
- Cross-agent synthesis for ripple effect detection
- Actionable fix outputs (inline diffs and Claude Code prompts)
- Support for Node.js/TypeScript, React, and .NET/C# projects

## Plugin Commands

### Code Review Commands

| Command | Description |
|---------|-------------|
| `/deep-code-review <file1> [file2...] [--output-file <path>]` | Deep review: Phase 1 (9 agents) → Phase 2 (5 gaps agents) → Synthesis (5 agents) |
| `/deep-code-review-staged [--output-file <path>]` | Deep review of staged git changes with full pipeline |
| `/quick-code-review <file1> [file2...] [--output-file <path>]` | Quick review (4 agents + 3 synthesis agents) |
| `/quick-code-review-staged [--output-file <path>]` | Quick review of staged git changes (7 agent invocations) |

### Documentation Review Commands

| Command | Description |
|---------|-------------|
| `/deep-docs-review [file1...] [--output-file <path>]` | Deep docs review: Phase 1 (6 agents) → Phase 2 (3 gaps agents) → Synthesis (4 agents) |
| `/quick-docs-review [file1...] [--output-file <path>]` | Quick docs review (4 agents + 3 synthesis agents) |

**Note:** All review commands also accept:
- `--language nodejs|react|dotnet` to force language detection
- `--prompt "<instructions>"` to pass additional instructions to agents
- `--skills <skill1,skill2,...>` to enable skill-informed orchestration (orchestrator interprets skills and generates tailored `skill_instructions` per agent)

## Plugin Skills

| Skill | Trigger Phrases |
|-------|-----------------|
| `reviewing-architecture-principles` | "check SOLID principles", "review SOLID", "find SOLID violations", "check DRY", "find code duplication", "find duplicate code", "check YAGNI", "find over-engineering", "check SoC", "separation of concerns", "mixed concerns", "file organization", "consolidate files", "architecture principles review" |
| `reviewing-bugs` | "find bugs", "check for bugs", "review for errors", "find logical errors", "check for null references", "find edge cases", "check for race conditions", "debug this code" |
| `reviewing-compliance` | "check CLAUDE.md compliance", "review against coding standards", "check AI agent instructions", "verify guidelines", "check coding conventions", "check naming conventions" |
| `reviewing-documentation` | "review documentation", "check docs", "audit README", "check CLAUDE.md", "verify AI instructions", "standardize docs", "review markdown", "check docs accuracy" |
| `reviewing-performance` | "check performance", "review for performance issues", "find slow code", "optimize", "check for memory leaks", "find N+1 queries", "check complexity", "profile code", "latency issues" |
| `reviewing-security` | "security review", "check for vulnerabilities", "audit security", "find security issues", "security scan", "check for injection", "find hardcoded secrets", "OWASP check" |
| `reviewing-technical-debt` | "find technical debt", "check for deprecated code", "find outdated patterns", "identify dead code", "check for workarounds", "find TODO comments", "assess code health", "code modernization review", "legacy code review" |

## Architecture

### Directory Structure

```
claude-code/plugins/code-review/
├── .claude-plugin/plugin.json       # Plugin metadata
├── commands/                        # Thin orchestration documents (reference shared/)
│   ├── deep-code-review.md          # Deep file review (19 agent invocations)
│   ├── deep-code-review-staged.md   # Deep staged review (19 agent invocations)
│   ├── deep-docs-review.md          # Deep documentation review (13 invocations)
│   ├── quick-code-review.md         # Quick file review (7 invocations)
│   ├── quick-code-review-staged.md  # Quick staged review (7 invocations)
│   └── quick-docs-review.md         # Quick documentation review (7 invocations)
├── agents/                          # Modular agent definitions (10 code + 6 docs agents)
│   ├── api-contracts-agent.md       # API compatibility
│   ├── architecture-agent.md        # Architecture patterns
│   ├── bug-detection-agent.md       # Logical errors & edge cases
│   ├── compliance-agent.md          # AI instructions compliance
│   ├── docs/                        # Documentation review agents (6 agents)
│   │   ├── accuracy-agent.md        # Code-doc sync, factual correctness
│   │   ├── clarity-agent.md         # Readability, jargon, audience
│   │   ├── completeness-agent.md    # Missing sections, coverage gaps
│   │   ├── consistency-agent.md     # Terminology, formatting, style
│   │   ├── examples-agent.md        # Code example validity
│   │   └── structure-agent.md       # Organization, links, AI instructions
│   ├── error-handling-agent.md      # Error handling gaps
│   ├── performance-agent.md         # Performance issues
│   ├── security-agent.md            # Security vulnerabilities
│   ├── synthesis-agent.md           # Cross-agent insights
│   ├── technical-debt-agent.md      # Technical debt detection
│   └── test-coverage-agent.md       # Test coverage gaps
├── skills/                          # Targeted review skills (progressive disclosure)
│   ├── reviewing-architecture-principles/
│   │   ├── SKILL.md
│   │   ├── references/
│   │   │   └── solid-dry-yagni-patterns.md
│   │   └── examples/
│   │       └── example-output.md
│   ├── reviewing-bugs/
│   │   ├── SKILL.md
│   │   ├── references/
│   │   │   └── common-bugs.md
│   │   └── examples/
│   │       └── example-output.md
│   ├── reviewing-compliance/
│   │   ├── SKILL.md
│   │   ├── references/
│   │   │   └── compliance-patterns.md
│   │   └── examples/
│   │       └── example-output.md
│   ├── reviewing-documentation/
│   │   ├── SKILL.md
│   │   ├── references/
│   │   │   ├── ai-instruction-templates.md
│   │   │   └── documentation-best-practices.md
│   │   └── examples/
│   │       └── example-output.md
│   ├── reviewing-performance/
│   │   ├── SKILL.md
│   │   ├── references/
│   │   │   └── performance-patterns.md
│   │   └── examples/
│   │       └── example-output.md
│   ├── reviewing-security/
│   │   ├── SKILL.md                 # Core skill instructions
│   │   ├── references/              # Detailed patterns (loaded on-demand)
│   │   │   └── common-vulnerabilities.md
│   │   └── examples/                # Sample output format
│   │       └── example-output.md
│   └── reviewing-technical-debt/
│       ├── SKILL.md
│       ├── references/
│       │   └── technical-debt-patterns.md
│       └── examples/
│           └── example-output.md
├── languages/                       # Language-specific configs
│   ├── dotnet.md                    # .NET/C# checks
│   ├── nodejs.md                    # Node.js/TypeScript checks
│   └── react.md                     # React checks (extends Node.js)
├── shared/
│   ├── orchestration-sequence.md    # Phase definitions and model selection (authoritative)
│   ├── agent-invocation-pattern.md  # Task invocation pattern for agents
│   ├── agent-common-instructions.md # Common MODE, false positives, gaps, pre-existing issue detection, output schema
│   ├── command-common-steps.md      # Common workflow steps for all commands
│   ├── settings-loader.md           # Settings loading and application
│   ├── file-processing.md           # File-based input validation and content gathering
│   ├── staged-processing.md         # Staged input validation and content gathering
│   ├── context-discovery.md         # Context discovery instructions
│   ├── skill-handling.md            # Skill resolution and orchestration (loaded when --skills used)
│   ├── synthesis-invocation-pattern.md # Synthesis agent task pattern
│   ├── validation-rules.md          # Validation process
│   ├── output-format.md             # Output templates, generation process (with fix_type)
│   ├── severity-definitions.md      # Severity classification
│   └── references/                  # Detailed reference content (progressive disclosure)
│       ├── complete-output-example.md # Complete output format example
│       ├── scope-determination.md   # Detailed scope options and edge cases
│       └── skill-troubleshooting.md # Common issues and solutions
├── templates/
│   └── code-review.local.md.example # Settings template for users
└── README.md
```

### Agent Configuration

See the following files for authoritative agent configuration:
- `shared/orchestration-sequence.md` - Model selection table, phase definitions, language-specific focus
- `shared/agent-invocation-pattern.md` - Task invocation template
- `shared/agent-common-instructions.md` - Common MODE, false positives, gaps behavior, pre-existing issue detection

### Agent Colors

Each agent has a unique color for visual identification during parallel execution:

#### Code Review Agents (10)

| Agent | Color |
|-------|-------|
| api-contracts-agent | cyan |
| architecture-agent | yellow |
| bug-detection-agent | red |
| compliance-agent | blue |
| error-handling-agent | orange |
| performance-agent | green |
| security-agent | purple |
| synthesis-agent | white |
| technical-debt-agent | brown |
| test-coverage-agent | white |

#### Documentation Review Agents (6)

| Agent | Color |
|-------|-------|
| accuracy-agent | red |
| clarity-agent | cyan |
| completeness-agent | green |
| consistency-agent | blue |
| examples-agent | yellow |
| structure-agent | purple |

**Color Assignment Rules:**

There are more agents than available colors. When assigning colors:

1. **Do not change existing colors** - Existing agent colors should not be changed unless absolutely necessary
2. **Minimize conflicts within each Phase** - Agents running in parallel within the same phase should have distinct colors when possible
3. **Repeating across Phases is acceptable** - Colors can be reused in different phases since they run sequentially
4. **Use white for overflow** - When colors must repeat within a phase, use white

**Current Color Usage:**
- Phase 1 code review agents have unique colors except synthesis and test-coverage (both white)
- Phase 2 reuses colors from Phase 1 (runs sequentially after Phase 1 completes)
- Synthesis phase runs 5 parallel instances of synthesis-agent, all using white (same agent type)
- Documentation agents reuse code agent colors since they run in separate pipelines (docs-review commands vs code-review commands)

### Deep Review Pipeline

1. **Phase 1** (9 agents parallel): Thorough mode review (5 Opus, 4 Sonnet)
2. **Phase 2** (5 Sonnet agents parallel): Gaps mode with Phase 1 findings as context
3. **Synthesis** (5 agents parallel): Cross-cutting concern detection
4. **Validation**: All issues validated before output

### MODE Parameter

- **thorough**: Comprehensive review, check all issues
- **gaps**: Focus on subtle issues, receives prior findings to skip duplicates (uses Sonnet for cost efficiency)
- **quick**: Fast pass on critical issues only

### Actionable Fix Formats

- **fix_type: diff**: Inline code diff for single-location fixes ≤10 lines
- **fix_type: prompt**: Claude Code prompt for multi-location/structural fixes

### Key Files

- `agents/*.md` - Individual agent definitions with MODE support
- `languages/*.md` - Language-specific checks and patterns
- `commands/*.md` - Thin orchestration documents referencing shared components
- `shared/orchestration-sequence.md` - Phase definitions, model selection table (authoritative)
- `shared/agent-invocation-pattern.md` - Task tool invocation template
- `shared/agent-common-instructions.md` - Common agent instructions (MODE, false positives, gaps, pre-existing issue detection, output schema)
- `shared/command-common-steps.md` - Common workflow steps shared by all commands
- `shared/settings-loader.md` - Settings loading and application
- `shared/file-processing.md` - Input validation and content gathering for file-based commands
- `shared/staged-processing.md` - Input validation and content gathering for staged commands
- `shared/context-discovery.md` - AI Agent Instructions and project type detection
- `shared/skill-handling.md` - Skill resolution and orchestration (lazy-loaded when --skills used)
- `shared/synthesis-invocation-pattern.md` - Synthesis agent invocation template
- `shared/validation-rules.md` - Issue validation process
- `shared/output-format.md` - Output formatting, templates, and generation process (with fix_type)
- `shared/severity-definitions.md` - Severity classification (Critical, Major, Minor, Suggestion)
- `skills/*/SKILL.md` - Skill definitions
- `templates/code-review.local.md.example` - User settings template

### Project Settings

Users can customize plugin behavior per-project by creating `.claude/code-review.local.md`:

```yaml
---
enabled: true
output_dir: "."
skip_agents: []
min_severity: "suggestion"
language: ""
---

# Project-specific instructions for review agents
```

See `shared/settings-loader.md` for loading logic and `README.md` for full documentation.

## Language Detection

- **Node.js/TypeScript**: Detected by `package.json`
- **React**: Detected by `react` or `react-dom` in package.json dependencies (extends Node.js checks)
- **.NET/C#**: Detected by `*.csproj`, `*.sln`, or `*.slnx`

## Making Changes

When modifying the plugin:

1. **Agent behavior**: Edit agent files in `agents/` directory
2. **Language-specific checks**: Edit files in `languages/` directory
3. **Validation rules**: Edit `shared/validation-rules.md`
4. **Output format/generation**: Edit `shared/output-format.md`
5. **Orchestration sequence**: Edit `shared/orchestration-sequence.md` (phase definitions, model selection, language-specific focus)
6. **Agent invocation pattern**: Edit `shared/agent-invocation-pattern.md`
7. **Common agent instructions**: Edit `shared/agent-common-instructions.md` (MODE, false positives, gaps, pre-existing issue detection)
8. **Common command steps**: Edit `shared/command-common-steps.md` (settings, context, validation, output)
9. **Common skill steps**: Skill workflows are self-contained in each `skills/*/SKILL.md`
10. **Command arguments**: Edit command YAML frontmatter in `commands/`
11. **Skills**: Edit skill files in `skills/*/SKILL.md`
12. **Skill references**: Add detailed patterns to `skills/*/references/`
13. **Skill examples**: Add sample outputs to `skills/*/examples/`
14. **Settings options**: Edit `shared/settings-loader.md` and `templates/code-review.local.md.example`

## Coding Conventions

### Alphabetical Ordering

**REQUIRED:** Always list agents, categories, and similar items in alphabetical order to avoid preferential treatment or implicit assumptions.

Apply alphabetical ordering to:
- Agent listings in orchestration documents (e.g., `orchestration-sequence.md`)
- Category pairs in synthesis configurations (order by first category)
- Tables with agent/skill names (e.g., Agent Colors table, Model Selection table)
- Bullet lists of agents or categories
- YAML keys within related_findings and similar structures
- Directory structure listings in documentation

**Examples:**

Agent listings:
```
# Correct (alphabetical)
- Launch: api-contracts, architecture, bug-detection, compliance, error-handling, performance, security, technical-debt, test-coverage

# Incorrect (not alphabetical)
- Launch: bug, performance, security, technical-debt, api, architecture, compliance, error-handling, test-coverage
```

Category pairs (alphabetize by first category):
```
# Correct
- Pairs: Architecture+Test Coverage, Bugs+Error Handling, Compliance+Technical Debt, Performance+Security

# Incorrect
- Pairs: Security+Performance, Architecture+Test Coverage, Bugs+Error Handling, Compliance+Bugs
```

**Rationale:** Alphabetical ordering ensures consistent, neutral presentation without implying priority or importance. It also makes it easier to find specific items in long lists.

### Step Numbering

**REQUIRED:** All workflow steps start at 1, not 0.

This applies to:
- Command workflows in `commands/*.md`
- Agent workflows in `agents/*.md`
- Skill workflows in `skills/*/SKILL.md`
- Shared step definitions in `shared/command-common-steps.md`

**Rationale:** Human documentation standards and consistency with agent workflows. All 16 agents already use Step 1 as their first step.

**Shared steps (in command-common-steps.md):**
- Steps 1, 2, 4, 6: Pre-review setup (methodology, settings, context, skills)
- Step 8.5: Usage tracking verification
- Steps 9-12: Post-review (validation, aggregation, output, write)

**Command-specific steps (inline in each command):**
- Steps 3, 5, 7-8: Input validation, content gathering, review execution

### File Path References

Plugin files use two distinct path reference patterns based on the official Anthropic skill best practices:

**1. Cross-Plugin References (to `shared/`, `languages/`, `agents/`)**

Use `${CLAUDE_PLUGIN_ROOT}` for references that cross skill/component boundaries:

```markdown
See `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md` for validation rules.
See `${CLAUDE_PLUGIN_ROOT}/languages/nodejs.md#security` for Node.js checks.
See `${CLAUDE_PLUGIN_ROOT}/agents/security-agent.md` for agent definition.
```

**2. Intra-Skill References (local `references/` and `examples/`)**

Use relative paths for references within a skill's own directory structure:

```markdown
# In skills/reviewing-security/SKILL.md
See `references/common-vulnerabilities.md` for vulnerability patterns.
See `examples/example-output.md` for sample output format.
```

**Rationale:** This follows the official Anthropic skill authoring best practices which shows relative paths for intra-skill references:

> "**Pattern 1: High-level guide with references**
> `**Form filling**: See [FORMS.md](FORMS.md) for complete guide`
> Claude loads FORMS.md, REFERENCE.md, or EXAMPLES.md only when needed."

**Do NOT convert intra-skill relative paths to `${CLAUDE_PLUGIN_ROOT}` paths** - this would break the documented progressive disclosure pattern.

## Skill Structure (Progressive Disclosure)

Each skill follows progressive disclosure pattern:

1. **SKILL.md** (~95-125 lines): Self-contained workflow loaded when skill triggers
2. **references/** (optional): Detailed patterns loaded on-demand
3. **examples/** (optional): Sample output formats for reference

```
skills/skill-name/
├── SKILL.md              # Always loaded when skill triggers (~130-170 lines)
├── references/           # Loaded when Claude needs detailed patterns
│   └── patterns.md
└── examples/             # Loaded when Claude needs output format reference
    └── example-output.md
```

Each skill is self-contained with its own workflow procedures (determining scope, gathering context, validating findings, and reporting results) to minimize file lookups during execution.

## Skill-Informed Orchestration

When `--skills` is provided to review commands, the orchestrator (running as Opus) interprets skill content and makes orchestration decisions:

1. **Skill Resolution**: Skills are resolved to SKILL.md files
2. **Structured Parsing**: Skills are parsed into structured data (focus_categories, auto_validated_patterns, false_positive_rules, methodology)
3. **Agent-Specific Instructions**: The orchestrator generates tailored `skill_instructions` per agent:
   - Review skills (reviewing-security, etc.) target their primary agent with checklists and focus areas
   - Methodology skills (superpowers:brainstorming, etc.) apply universally to all agents
4. **Validation Adjustments**: Auto-validated patterns skip validation; false positive rules filter findings
5. **Synthesis Adjustments**: Skill-specific cross-cutting questions may be added

See `shared/skill-handling.md` for implementation details.

**Skill Loading:** The orchestrator MUST use the Skill() tool to load skills. Direct file read is only used as fallback if Skill() tool fails.

## Version Management

### Release Process

When preparing a new release:

1. **Find changes since last release:**
   ```bash
   # List git tags to find previous version
   git tag -l --sort=-v:refname | head -5

   # View commits since last tag (replace <prev> with previous version)
   git log <prev>..HEAD --oneline -- claude-code/plugins/code-review/

   # View files changed
   git diff <prev>..HEAD --stat -- claude-code/plugins/code-review/
   ```

2. **Determine version bump:**
   - **Patch (3.0.x)**: Bug fixes, documentation updates, minor enhancements
   - **Minor (3.x.0)**: New features, new commands, new agents
   - **Major (x.0.0)**: Breaking changes, architecture overhauls

3. **Update CHANGELOG.md first:**
   - Add new version section at top (after header)
   - Document all Added, Changed, Fixed, Removed items
   - Follow [Keep a Changelog](https://keepachangelog.com/) format
   - **Base release notes on actual file changes (`git diff`), not commit messages** - commit messages may be incomplete or misleading

4. **Update version in all locations:**

   - `claude-code/plugins/code-review/.claude-plugin/plugin.json`
   - `.claude-plugin/marketplace.json` (repository root)
   - `claude-code/plugins/code-review/README.md` (Current Version line)

   > **Note:** Per Anthropic's plugin guidance, only `plugin.json` should contain the authoritative version. Individual agent, skill, and command files should NOT have version fields.

5. **Verify versions are consistent:**
   ```bash
   # Verify no old version remains (exclude CHANGELOG history)
   grep -r "<prev>" --include="*.md" --include="*.json" | grep -v CHANGELOG

   # Verify new version count (3: 1 plugin.json + 1 marketplace.json + 1 README)
   grep -r "<new>" --include="*.md" --include="*.json" | grep -v CHANGELOG | wc -l

   # Verify CHANGELOG has new section
   head -20 claude-code/plugins/code-review/CHANGELOG.md
   ```

6. **Commit changes:**
   ```bash
   git add -A
   git commit -m "Release v<new>: <summary of changes>"
   ```

7. **Create and push git tag:**
   ```bash
   git tag v<new>
   git push origin main
   git push origin v<new>
   ```
