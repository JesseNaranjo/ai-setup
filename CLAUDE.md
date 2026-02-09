# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Claude Code plugin repository containing the **Code Review Plugin** (v3.4.2) - a modular 17-agent architecture with:
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

| Skill | Focus |
|-------|-------|
| `reviewing-architecture-principles` | SOLID, DRY, YAGNI, SoC violations |
| `reviewing-bugs` | Logical errors, null references, edge cases, race conditions |
| `reviewing-compliance` | CLAUDE.md and coding standards compliance |
| `reviewing-documentation` | Documentation accuracy, structure, AI instructions |
| `reviewing-performance` | Performance issues, memory leaks, N+1 queries |
| `reviewing-security` | Vulnerabilities, injection, secrets, OWASP |
| `reviewing-technical-debt` | Deprecated code, dead code, outdated patterns |

## Architecture

### Directory Structure

```
claude-code/plugins/code-review/
├── .claude-plugin/plugin.json       # Plugin metadata
├── commands/                        # Self-contained orchestration documents (inline common steps, reference shared/)
│   ├── deep-code-review.md          # Deep file review (19 agent invocations)
│   ├── deep-code-review-staged.md   # Deep staged review (19 agent invocations)
│   ├── deep-docs-review.md          # Deep documentation review (13 invocations)
│   ├── quick-code-review.md         # Quick file review (7 invocations)
│   ├── quick-code-review-staged.md  # Quick staged review (7 invocations)
│   └── quick-docs-review.md         # Quick documentation review (7 invocations)
├── agents/                          # Modular agent definitions (10 code + 7 docs agents)
│   ├── code/                        # Code review agents (10 agents)
│   │   ├── api-contracts-agent.md   # API compatibility
│   │   ├── architecture-agent.md    # Architecture patterns
│   │   ├── bug-detection-agent.md   # Logical errors & edge cases
│   │   ├── compliance-agent.md      # AI instructions compliance
│   │   ├── error-handling-agent.md  # Error handling gaps
│   │   ├── performance-agent.md     # Performance issues
│   │   ├── security-agent.md        # Security vulnerabilities
│   │   ├── synthesis-code-agent.md  # Cross-agent insights (code reviews)
│   │   ├── technical-debt-agent.md  # Technical debt detection
│   │   └── test-coverage-agent.md   # Test coverage gaps
│   └── docs/                        # Documentation review agents (7 agents)
│       ├── accuracy-agent.md        # Code-doc sync, factual correctness
│       ├── clarity-agent.md         # Readability, jargon, audience
│       ├── completeness-agent.md    # Missing sections, coverage gaps
│       ├── consistency-agent.md     # Terminology, formatting, style
│       ├── examples-agent.md        # Code example validity
│       ├── structure-agent.md       # Organization, links, AI instructions
│       └── synthesis-docs-agent.md  # Cross-agent insights (docs reviews)
├── skills/                          # 7 review skills (progressive disclosure)
│   ├── reviewing-architecture-principles/
│   ├── reviewing-bugs/
│   ├── reviewing-compliance/
│   ├── reviewing-documentation/     # Has 2 reference files
│   ├── reviewing-performance/
│   ├── reviewing-security/          # Expanded example (all 7 skills follow this pattern):
│   │   ├── SKILL.md                 # Core skill instructions (~90-155 lines)
│   │   ├── references/              # Detailed patterns (loaded on-demand)
│   │   │   ├── common-vulnerabilities.md
│   │   │   └── owasp-reference.md
│   │   └── examples/                # Sample output format
│   │       └── example-output.md
│   └── reviewing-technical-debt/
├── languages/                       # Language-specific configs
│   ├── dotnet.md                    # .NET/C# checks
│   ├── nodejs.md                    # Node.js/TypeScript checks
│   └── react.md                     # React checks (extends Node.js)
├── shared/
│   ├── agent-common-instructions.md # Common MODE, false positives, language checks, output schema, compact severity definitions
│   ├── docs-processing.md           # Docs input validation and content gathering
│   ├── file-processing.md           # File-based input validation and content gathering
│   ├── output-format.md             # Output format specification (progressive disclosure, loaded at Steps 9-12)
│   ├── pre-review-setup.md          # Settings loading + context discovery (combined)
│   ├── review-orchestration-code.md # Code review: phases, model selection, invocation patterns, gaps mode behavior
│   ├── review-orchestration-docs.md # Docs review: phases, model selection, invocation patterns, gaps mode behavior
│   ├── skill-handling.md            # Skill resolution and orchestration (loaded when --skills used)
│   ├── staged-processing.md         # Staged input validation, content gathering, and pre-existing issue detection
│   └── references/                  # Detailed reference content (progressive disclosure)
│       ├── complete-output-example.md # Complete output format example
│       ├── lsp-integration.md       # LSP integration details for Node.js and .NET
│       ├── skill-troubleshooting.md # Common issues and solutions
│       ├── validation-rules-code.md # Code review validation, aggregation, auto-validation patterns, false positive rules
│       └── validation-rules-docs.md # Docs review validation, aggregation, auto-validation patterns, false positive rules
├── templates/
│   └── code-review.local.md.example # Settings template for users
└── README.md
```

### Agent Colors

Agent colors are defined in each agent's YAML frontmatter (`color: <color>`).

**Color Assignment Rules:**

There are more agents than available colors. When assigning colors:

1. **Do not change existing colors** - Existing agent colors should not be changed unless absolutely necessary
2. **Minimize conflicts within each Phase** - Agents running in parallel within the same phase should have distinct colors when possible
3. **Repeating across Phases is acceptable** - Colors can be reused in different phases since they run sequentially
4. **Use white for overflow** - When colors must repeat within a phase, use white

**Current Color Usage:**
- Phase 1 code review agents have unique colors except synthesis and test-coverage (both white)
- Phase 2 reuses colors from Phase 1 (runs sequentially after Phase 1 completes)
- Synthesis phase runs parallel instances of synthesis-code-agent or synthesis-docs-agent, all using white
- Documentation agents reuse code agent colors since they run in separate pipelines (docs-review commands vs code-review commands)

### Deep Review Pipeline

1. **Phase 1** (9 agents parallel): Thorough mode review (5 Opus, 4 Sonnet)
2. **Phase 2** (5 Sonnet agents parallel): Gaps mode with Phase 1 findings as context
3. **Synthesis** (5 agents parallel): Cross-cutting concern detection
4. **Validation**: All issues validated before output

### Gaps Mode Agent Selection Rationale

**Selected agents:** bug-detection, compliance, performance, security, technical-debt

**Rationale:**
1. **High complexity domains**: Security, performance, and bugs have many subtle edge cases that benefit from a second pass with fresh perspective
2. **Domain overlap potential**: Compliance and technical-debt often surface issues that other agents might frame differently
3. **Cost-benefit analysis**: These 5 agents provide the best coverage-to-cost ratio for gaps analysis

**Excluded agents:** api-contracts, architecture, error-handling, test-coverage
- These domains have fewer subtle edge cases that benefit from gaps pass
- Their issues are typically more binary (present/absent) rather than nuanced
- Architecture and API contracts are better caught in thorough mode or synthesis

### MODE Parameter

- **thorough**: Comprehensive review, check all issues
- **gaps**: Focus on subtle issues, receives prior findings to skip duplicates (uses Sonnet for cost efficiency)
- **quick**: Fast pass on critical issues only

### Actionable Fix Formats

- **fix_type: diff**: Inline code diff for single-location fixes ≤10 lines
- **fix_type: prompt**: Claude Code prompt for multi-location/structural fixes

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

See `shared/pre-review-setup.md` for loading logic and `README.md` for full documentation.

### Language Detection

- **Node.js/TypeScript**: Detected by `package.json`
- **React**: Detected by `react` or `react-dom` in package.json dependencies (extends Node.js checks)
- **.NET/C#**: Detected by `*.csproj`, `*.sln`, or `*.slnx`

### Skill Structure (Progressive Disclosure)

Each skill follows progressive disclosure: `SKILL.md` (~90-155 lines) is always loaded when triggered; `references/` and `examples/` subdirectories are loaded on-demand. Skills are self-contained with their own workflow procedures.

### Skill-Informed Orchestration

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

## Making Changes

When modifying the plugin:

1. **Agent behavior**: Edit agent files in `agents/code/` or `agents/docs/`
2. **Language-specific checks**: Edit files in `languages/` directory
3. **Validation rules (code)**: Edit `shared/references/validation-rules-code.md` (includes validation process, aggregation, auto-validation patterns, false positive rules)
4. **Validation rules (docs)**: Edit `shared/references/validation-rules-docs.md` (includes validation process, aggregation, auto-validation patterns, false positive rules)
5. **Output format/generation**: Edit `shared/output-format.md`
6. **Severity definitions**: Edit `shared/agent-common-instructions.md` "Severity Definitions" section
7. **Code review orchestration**: Edit `shared/review-orchestration-code.md` (phases, model selection, invocation patterns, language-specific focus, gaps mode behavior)
8. **Docs review orchestration**: Edit `shared/review-orchestration-docs.md` (phases, model selection, invocation patterns, gaps mode behavior)
9. **Common agent instructions**: Edit `shared/agent-common-instructions.md` (MODE, false positives, language checks)
10. **Pre-existing issue detection**: Edit `shared/staged-processing.md` "Pre-Existing Issue Detection" section
11. **Skills**: Edit `skills/*/SKILL.md` for self-contained workflows, add patterns to `references/`, examples to `examples/`
12. **Command arguments**: Edit command YAML frontmatter in `commands/`
13. **Settings options**: Edit `shared/pre-review-setup.md` and `templates/code-review.local.md.example`

## Coding Conventions

### Instruction Quality Preservation

**REQUIRED:** When modifying instruction content (agents, skills, shared files, commands), preserve the quality and precision of existing guidance.

- **Preserve specificity**: Never replace concrete patterns, examples, or exact checks with vague or generalized guidance
- **Preserve rationale**: Keep "why" explanations alongside rules — these prevent future contributors from removing seemingly arbitrary constraints
- **Preserve calibration**: Do not change severity thresholds, false positive rules, or validation patterns without explicit request
- **No lossy compression**: When consolidating, moving, or restructuring content, diff before/after to confirm no meaningful guidance was dropped
- **No unsolicited cleanup**: Do not reword, simplify, or "improve" instruction prose that wasn't part of the requested change

**Rationale:** The instruction files in this repo are LLM prompts whose effectiveness depends on precise wording, specific examples, and calibrated thresholds. Generic editing instincts (simplify, shorten, generalize) directly degrade agent performance.

### Content Categories

**CRITICAL:** Every file in this plugin belongs to exactly one of three categories. Never place content in a directory belonging to a different category. This is the most consequential convention in this file — violating it silently degrades review performance by polluting the Opus orchestrator's context window with irrelevant content.

| Category | Directories | Consumed By | When |
|----------|-------------|-------------|------|
| **Development-time guidance** | `CLAUDE.md` | Claude Code | When authoring/modifying the plugin |
| **Plugin runtime content** | `commands/`, `agents/`, `shared/`, `skills/`, `languages/` | Opus orchestrator + agents | During every review execution |
| **User-facing documentation** | `README.md`, `CHANGELOG.md`, `templates/` | Human users | When configuring or understanding the plugin |

**The core rule:** Runtime directories (`commands/`, `agents/`, `shared/`, `skills/`, `languages/`) are loaded into the Opus orchestrator's context window during review execution. Every token has a direct cost on every review. **Never place development-time authoring guidance or documentation into runtime directories.**

**Common violation:** The instinct to "extract" CLAUDE.md content into `shared/references/` moves tokens from a cheap context (loaded once per authoring session) into an expensive context (loaded on every review). Ask "who consumes this content and when?" — if a developer modifying the plugin, it belongs in `CLAUDE.md`.

### Alphabetical Ordering

**REQUIRED:** Always list agents, categories, and similar items in alphabetical order to avoid preferential treatment or implicit assumptions.

Apply alphabetical ordering to:
- Agent listings in orchestration documents (e.g., `review-orchestration-code.md`)
- Category pairs in synthesis configurations (order by first category)
- Tables with agent/skill names (e.g., Model Selection table)
- Bullet lists of agents or categories
- YAML keys within related_findings and similar structures
- Directory structure listings in documentation

**Example:**
```
# Correct (alphabetical)
- Launch: api-contracts, architecture, bug-detection, compliance, error-handling, performance, security, technical-debt, test-coverage
```

**Rationale:** Alphabetical ordering ensures consistent, neutral presentation without implying priority or importance. It also makes it easier to find specific items in long lists.

### Step Numbering

**REQUIRED:** All workflow steps start at 1, not 0.

This applies to:
- Command workflows in `commands/*.md`
- Agent workflows in `agents/code/*.md` and `agents/docs/*.md`
- Skill workflows in `skills/*/SKILL.md`

**Rationale:** Human documentation standards and consistency with agent workflows. All 17 agents already use Step 1 as their first step.

**Step layout in commands:**
- Steps 1, 2, 4, 6: Pre-review setup (methodology, settings, context, skills) - inlined in each command
- Steps 3, 5, 7-8: Input validation, content gathering, review execution - command-specific
- Steps 9-12: Post-review (validation, aggregation, output, write) - inlined in each command

### Command Step Inlining

Each command file inlines its pre-review setup and post-review steps directly rather than referencing a shared common-steps file. The 4 code review commands share ~60 lines of identical inlined content; the 2 docs review commands use a different layout.

**Rationale:** A shared `command-common-steps.md` was tried but created a second level of indirection (commands → common-steps → shared files), adding an extra file to the Opus context window. Inlining keeps commands self-contained with one-hop references to shared files.

**Revisit if:** command count exceeds 6, shared steps diverge significantly, or a way is found to share steps without extra Opus context files.

### Commands Directory

Commands remain in `commands/` despite Anthropic's Plugin Reference labeling it as "legacy; use `skills/` for new skills." The command YAML frontmatter uses fields (`allowed-tools`, `argument-hint`, `model`) that are command-specific and not part of the skill YAML schema (`name`, `description` only). These are complex orchestration entry points, not simple trigger-response commands. Migrating would break orchestrator invocation for zero context reduction.

### File Path References

Plugin files use two distinct path reference patterns:

**1. Cross-Plugin References (to `shared/`, `languages/`, `agents/`)** — use `${CLAUDE_PLUGIN_ROOT}`:

```markdown
See `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md` for validation rules.
See `${CLAUDE_PLUGIN_ROOT}/agents/code/security-agent.md` for agent definition.
```

**2. Intra-Skill References (local `references/` and `examples/`)** — use relative paths:

```markdown
# In skills/reviewing-security/SKILL.md
See `references/common-vulnerabilities.md` for vulnerability patterns.
```

**Rationale:** Follows Anthropic skill authoring best practices — relative paths enable progressive disclosure where Claude loads reference files only when needed.

**Do NOT convert intra-skill relative paths to `${CLAUDE_PLUGIN_ROOT}` paths** — this would break the documented progressive disclosure pattern.

## Version Management

### Release Process

When preparing a new release:

1. **Find changes since last release:**
   ```bash
   git tag -l --sort=-v:refname | head -5
   git log <prev>..HEAD --oneline -- claude-code/plugins/code-review/
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
   - **Base release notes on actual file changes (`git diff`), not commit messages**

4. **Update version in all locations:**
   - `claude-code/plugins/code-review/.claude-plugin/plugin.json`
   - `.claude-plugin/marketplace.json` (repository root)
   - `claude-code/plugins/code-review/README.md` (Current Version line)

   > **Note:** Per Anthropic's plugin guidance, only `plugin.json` should contain the authoritative version. Individual agent, skill, and command files should NOT have version fields.

5. **Verify, commit, and tag:**
   ```bash
   grep -r "<prev>" --include="*.md" --include="*.json" | grep -v CHANGELOG
   grep -r "<new>" --include="*.md" --include="*.json" | grep -v CHANGELOG | wc -l  # expect 3
   head -20 claude-code/plugins/code-review/CHANGELOG.md
   git add -A && git commit -m "Release v<new>: <summary of changes>"
   git tag v<new> && git push origin main && git push origin v<new>
   ```
