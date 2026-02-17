# CLAUDE.md

This file and nearly all content in this repository is written for LLM consumption, not humans. This file provides guidance to Claude Code when working with code in this repository. All other plugin files (`commands/`, `agents/`, `shared/`, `skills/`, `languages/`, `templates/`) are LLM prompts executed by the Opus orchestrator and review agents at runtime.

The only human-readable files are:
- `README.md` and `CHANGELOG.md`
- `.claude-plugin/marketplace.json` (`name` and `description` fields only)
- `claude-code/plugins/code-review/.claude-plugin/plugin.json` (`name` and `description` fields only)

Do not optimize any other file for human readability. Do not add prose softeners, transitional phrases, or explanatory filler that aids human reading but adds tokens to LLM context windows.

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
├── commands/                        # Orchestration entry points (inline steps, reference shared/; see "Commands Directory" convention)
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
│   │   └── references/              # Detailed patterns (loaded on-demand)
│   │       └── common-vulnerabilities.md
│   └── reviewing-technical-debt/
├── languages/                       # Language-specific configs
│   ├── dotnet.md                    # .NET/C# checks
│   ├── nodejs.md                    # Node.js/TypeScript checks
│   └── react.md                     # React checks (extends Node.js)
├── shared/
│   ├── docs-processing.md           # Docs input validation and content gathering
│   ├── example-output.md            # Shared example output (referenced by all 7 skills)
│   ├── file-processing.md           # File-based input validation and content gathering
│   ├── output-format.md             # Output format specification (progressive disclosure, loaded at Steps 9-12)
│   ├── pre-review-setup.md          # Settings loading + context discovery (combined)
│   ├── review-orchestration-code.md # Code review: phases, model selection, invocation patterns, gaps mode behavior, agent common instructions, category FP rules
│   ├── review-orchestration-docs.md # Docs review: phases, model selection, invocation patterns, gaps mode behavior, agent common instructions, category FP rules
│   ├── review-validation-code.md    # Code validation: batch validation, aggregation, auto-validation patterns
│   ├── review-validation-docs.md    # Docs validation: batch validation, aggregation, auto-validation patterns
│   ├── skill-handling.md            # Skill resolution and orchestration (loaded when --skills used)
│   ├── staged-processing.md         # Staged input validation, content gathering, and pre-existing issue detection
│   └── references/                  # LSP integration details (progressive disclosure)
│       └── lsp-integration.md       # LSP integration details for Node.js and .NET
├── templates/
│   └── code-review.local.md.example # Settings template (parsed by orchestrator at runtime)
└── README.md
```

### Agent Frontmatter

All 17 agents use the same YAML frontmatter fields:

```yaml
---
name: <domain>-agent           # Required
description: <role+trigger>    # Required. "[Role] specialist. Use [for/when] [triggers]."
color: <color>                 # See color rules below
tools: ["Read", "Grep", "Glob"]
---
```

Other Plugin Reference fields (`disallowedTools`, `permissionMode`, `maxTurns`, `skills`, `mcpServers`, `hooks`, `memory`) are not used.

**Color rules:** Minimize conflicts within each parallel phase; reuse across sequential phases is fine. Use `white` for overflow. Do not change existing colors without necessity.

### Deep Review Pipeline

1. **Phase 1** (9 agents parallel): Thorough mode review (5 Opus, 4 Sonnet)
2. **Phase 2** (5 Sonnet agents parallel): Gaps mode with Phase 1 findings as context
3. **Synthesis** (5 agents parallel): Cross-cutting concern detection (requires findings in BOTH input categories; single-category insights are rejected)
4. **Validation**: All issues validated before output

### Agent Content Distribution

The orchestration files (`review-orchestration-code.md`, `review-orchestration-docs.md`) contain inlined agent common instructions (MODE, false positives, output schema) and category-specific false positive rules. The orchestrator distributes relevant portions to each agent via `additional_instructions`, along with language-specific checks from `languages/*.md` and LSP diagnostic codes (when LSP is available). Agents do not read these shared files directly.

Validation and auto-validation content is in separate files (`review-validation-code.md`, `review-validation-docs.md`), loaded on-demand at Steps 9-12 (post-review). This progressive disclosure keeps validation content out of the Opus context during the expensive Steps 7-8 phase.

See `shared/review-orchestration-code.md` "Agent Common Content Distribution" and `shared/review-orchestration-docs.md` for implementation details.

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

Each skill follows progressive disclosure: `SKILL.md` is always loaded when triggered; `references/` subdirectories are loaded on-demand (one level deep from SKILL.md). All 7 skills share a single example output file at `shared/example-output.md` (referenced via `${CLAUDE_PLUGIN_ROOT}`). Skills are self-contained with their own workflow procedures. Review skills provide unique value (scope prioritization, FP adjustments, reference files, methodology) but do not duplicate agent category checklists — categories are in agent files only.

**Description patterns:**
- Skills: `"[What in third person]. Use when [specific triggers]."` — e.g., `"Detects injection attacks... Use when checking for security vulnerabilities during code review."`
- Agents: `"[Role] specialist. Use [for] [triggers]."` — e.g., `"Security vulnerability specialist. Use for detecting injection attacks..."`

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

### Agent & Orchestration
- **Agent behavior**: Edit `agents/code/` or `agents/docs/`
- **Code review orchestration**: Edit `shared/review-orchestration-code.md` (phases, model selection, invocation patterns, language-specific focus, gaps mode behavior, agent common instructions, category-specific FP rules)
- **Docs review orchestration**: Edit `shared/review-orchestration-docs.md` (phases, model selection, invocation patterns, gaps mode behavior, agent common instructions, category-specific FP rules)

### Validation & Output
- **Validation rules (code)**: Edit `shared/review-validation-code.md` (validation process, aggregation, auto-validation patterns)
- **Validation rules (docs)**: Edit `shared/review-validation-docs.md` (validation process, aggregation, auto-validation patterns)
- **Output format/generation**: Edit `shared/output-format.md`
- **Severity definitions**: Each agent defines calibrated thresholds in its own file under `agents/code/` or `agents/docs/`

### Skills & Language
- **Skills**: Edit `skills/*/SKILL.md`; add patterns to `references/`; shared example at `shared/example-output.md`
- **Language-specific checks**: Edit files in `languages/`

### Commands & Settings
- **Command arguments**: Edit command YAML frontmatter in `commands/`
- **Settings options**: Edit `shared/pre-review-setup.md` and `templates/code-review.local.md.example`
- **Pre-existing issue detection**: Edit `shared/staged-processing.md` "Pre-Existing Issue Detection" section

## Coding Conventions

### Instruction Quality Preservation

**REQUIRED:** When modifying instruction content (CLAUDE.md, agents, skills, shared files, commands), preserve the quality and precision of existing guidance.

- **Preserve specificity**: Never replace concrete patterns, examples, or exact checks with vague or generalized guidance
- **Preserve rationale**: Keep "why" explanations alongside rules — these prevent future contributors from removing seemingly arbitrary constraints
- **Preserve calibration**: Do not change severity thresholds, false positive rules, or validation patterns without explicit request
- **No lossy compression**: When consolidating, moving, or restructuring content, diff before/after to confirm no meaningful guidance was dropped
- **No unsolicited cleanup**: Do not reword, simplify, or "improve" instruction prose that wasn't part of the requested change

**Rationale:** The instruction files in this repo are LLM prompts whose effectiveness depends on precise wording, specific examples, and calibrated thresholds. Generic editing instincts (simplify, shorten, generalize) directly degrade agent performance.

### Agent Checklist Compression

Model-aware compression of thorough-mode checklists:
- **Opus agents**: Compress to high-level triggers (1-3 lines). "Claude is already smart" — only add domain context Claude doesn't already have.
- **Sonnet agents**: Moderate compression. Keep domain-specific guidance, items with calibrated thresholds, and unique methodology. Agents with only meta-rules (compliance) need no compression.
- **All agents**: Preserve gaps/quick mode items, severity thresholds, output schema verbatim. MODE differentiation (thorough/gaps/quick sections) intact.

### Content Audience

**CRITICAL:** Nearly all content in this repository is written for LLM consumption. Do not adjust content for human readability unless the file is explicitly listed as human-readable below.

**LLM-consumed content** (everything except the files listed below):

| Subcategory | Files | Consumer | When |
|-------------|-------|----------|------|
| Development-time | `CLAUDE.md` | Claude Code | When authoring/modifying the plugin |
| Runtime | `commands/`, `agents/`, `shared/`, `skills/`, `languages/`, `templates/` | Opus orchestrator + agents | During every review execution |

**Human-readable files** (exhaustive list):
- `README.md` — plugin documentation for users
- `CHANGELOG.md` — release notes for users
- `.claude-plugin/marketplace.json` — `name` and `description` fields only
- `claude-code/plugins/code-review/.claude-plugin/plugin.json` — `name` and `description` fields only

**Everything not listed above is an LLM prompt.** This includes `templates/` (the settings format is parsed by the Opus orchestrator at runtime).

**Anti-drift rules:**
- Do not add prose transitions, softening language, or filler that aids human scanning but adds tokens
- Do not cite "human documentation standards" as rationale for LLM prompt conventions
- Do not use "developer" or "user" when the actual consumer is Claude Code or the Opus orchestrator
- When unsure about audience, default to LLM-optimized — the human-readable list above is exhaustive

**The core rule:** Runtime files are loaded into the Opus orchestrator's context window during every review. Every token has a direct cost. Never place development-time guidance into runtime directories.

**Common violation:** The instinct to "extract" CLAUDE.md content into `shared/references/` moves tokens from a cheap context (loaded once per authoring session) into an expensive context (loaded on every review). Ask "who consumes this content and when?" — if Claude Code when modifying the plugin, it belongs in `CLAUDE.md`.

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

**Rationale:** Consistency with agent workflows. All 17 agents already use Step 1 as their first step.

**Step layout in commands:**
- Steps 1, 2, 4, 6: Pre-review setup (methodology, settings, context, skills) - inlined in each command
- Steps 3, 5: Input validation, content gathering - command-specific
- Steps 7-8: Review execution, synthesis - references to orchestration files
- Steps 9-12: Post-review (validation, aggregation, output, write) - compressed inline

### Command Step Inlining

Each command file inlines its pre-review setup steps (Steps 1-6) directly rather than referencing a shared common-steps file. Steps 7-8 reference orchestration files for review execution and synthesis. Steps 9-12 are compressed inline. The 4 code review commands share ~40 lines of identical inlined content; the 2 docs review commands use a different layout.

**Rationale:** A shared `command-common-steps.md` was tried but created a second level of indirection (commands → common-steps → shared files), adding an extra file to the Opus context window. Inlining keeps commands self-contained with one-hop references to shared files.

**Revisit if:** command count exceeds 6, shared steps diverge significantly, or a way is found to share steps without extra Opus context files.

### Intentional Cross-File Duplication

| File Pair | Shared Content | ~Lines | Rationale |
|-----------|---------------|--------|-----------|
| `review-orchestration-code.md` / `review-orchestration-docs.md` | File Entry Schema, Agent Common Instructions, Gaps Mode core | ~70 | Only one loaded per execution; extracting adds a file read |
| `review-validation-code.md` / `review-validation-docs.md` | Batch Validation, Validator Schema, Common FP, Verdicts, Aggregation | ~70 | Same rationale: only one loaded per execution |

**Maintenance rule:** When modifying shared content in one file, `grep -r` for the same section heading in the paired file and update both.

### Commands Directory

Commands remain in `commands/` despite Anthropic's Plugin Reference labeling it as "legacy; use `skills/` for new skills." The command YAML frontmatter uses fields (`allowed-tools`, `argument-hint`, `model`) that are command-specific and not part of the skill YAML schema (`name`, `description` only). These are complex orchestration entry points, not simple trigger-response commands. Migrating would break orchestrator invocation for zero context reduction.

**Command frontmatter fields:** `name`, `allowed-tools`, `description`, `argument-hint`, `model`. All commands use `model: opus`. The `allowed-tools` list includes `Task` (agent invocation), git-scoped `Bash` patterns, `Read`, `Write`, `Glob`.

### File Path References

Plugin files use two distinct path reference patterns:

**1. Cross-Plugin References (to `shared/`, `languages/`, `agents/`)** — use `${CLAUDE_PLUGIN_ROOT}`:

```markdown
See `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md` for validation rules.
See `${CLAUDE_PLUGIN_ROOT}/agents/code/security-agent.md` for agent definition.
```

**2. Intra-Skill References (local `references/`)** — use relative paths:

```markdown
# In skills/reviewing-security/SKILL.md
See `references/common-vulnerabilities.md` for vulnerability patterns.
```

**Rationale:** Follows Anthropic skill authoring best practices — relative paths enable progressive disclosure where Claude loads reference files only when needed.

**Do NOT convert intra-skill relative paths to `${CLAUDE_PLUGIN_ROOT}` paths** — this would break the documented progressive disclosure pattern.

**Exception:** The shared example output (`shared/example-output.md`) uses `${CLAUDE_PLUGIN_ROOT}` because it's shared infrastructure, not a skill-local file.

## Common Gotchas

- **Trailing newlines**: Subagent file rewrites often drop trailing newlines. Verify with `tail -c 1` after bulk operations.
- **Reference integrity**: When moving content between files, `grep -r "section name"` before AND after to catch all references (commands, skills, shared files).
- **Content category violations**: Before adding content to `shared/` or `shared/references/`, ask "who consumes this?" If the answer is "Claude Code when modifying the plugin," it belongs in CLAUDE.md, not runtime directories.
- **Synthesis constraint**: Synthesis insights require findings in BOTH input categories. Single-category insights are rejected at validation.
- **Validation progressive disclosure**: Orchestration files (`review-orchestration-*.md`) cross-reference validation files (`review-validation-*.md`). Do not re-merge them — the split reduces Opus context by ~53% during Steps 7-8.

### Auto-Validation Pattern Design Notes

These notes apply when modifying patterns in `review-validation-{code|docs}.md`:
- Patterns are case-insensitive for SQL keywords
- Patterns require at least one character (`[^'"]+`) to avoid matching empty string placeholders like `password = ""`
- Patterns check for common variable names indicating user input: `req`, `request`, `params`, `query`, `body`, `input`, `user`
- Empty catch pattern allows a single comment line to avoid flagging intentional empty catches with explanation

### Content Strategy Rationale

Agents have access to Read, Grep, and Glob tools. For Phase 2 (gaps) and Synthesis, providing file paths instead of full content allows agents to fetch content on-demand, reducing baseline token usage while preserving capability. Phase 1 provides full file content and diffs; Phase 2 provides diffs only; Synthesis provides file paths only. The operational details are in the Review Sequence sections of `review-orchestration-code.md`.

## Version Management

**Version bump rules:** Patch = fixes/docs. Minor = new features/agents/commands. Major = breaking changes.

**Version locations** (all three must match): `plugin.json`, `marketplace.json` (repo root), `README.md` ("Current Version"). Per Anthropic guidance, only `plugin.json` is authoritative — individual agent/skill/command files must NOT have version fields.

**Release steps:**
1. Find previous tag and diff changes: `git tag -l --sort=-v:refname | head -5` then `git log <prev>..HEAD --oneline -- claude-code/plugins/code-review/`
2. Update CHANGELOG.md first (Keep a Changelog format, base on `git diff` not commit messages)
3. Update version in all 3 locations
4. Verify: `grep -r "<prev>" --include="*.md" --include="*.json" | grep -v CHANGELOG` (expect 0) and `grep -r "<new>" ... | wc -l` (expect 3)
5. Commit, tag, push: `git tag v<new> && git push origin main && git push origin v<new>`
