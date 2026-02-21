# CLAUDE.md

This file and nearly all content in this repository is written for LLM consumption, not humans. This file provides guidance to Claude Code when working with code in this repository. All other plugin files (`agents/`, `shared/`, `skills/`, `languages/`, `templates/`) are LLM prompts executed by the Opus orchestrator and review agents at runtime.

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

### Review Commands

| Command | Description |
|---------|-------------|
| `/code-review [<file1> [file2...] \| --staged] [--depth deep\|quick] [--output-file <path>]` | Code review for files or staged changes with configurable depth (deep: up to 19 agents, quick: up to 7) |
| `/docs-review [file1...] [--depth deep\|quick] [--output-file <path>]` | Docs review with configurable depth (deep: up to 13 agents, quick: up to 7) |

**Note:** All review commands also accept:
- `--language nodejs|react|dotnet` to force language detection (code reviews only)
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
├── skills/                          # 11 skills (2 orchestration + 7 review + 2 internal)
│   ├── agent-review-instructions/   # Internal: MODE, FP rules, output schema (loaded by agents via skills field)
│   │   └── SKILL.md
│   ├── code-review/                 # Orchestration: code review pipeline (deep: up to 19, quick: up to 7 invocations)
│   │   └── SKILL.md
│   ├── docs-review/                 # Orchestration: docs review pipeline (deep: up to 13, quick: up to 7 invocations)
│   │   └── SKILL.md
│   ├── reviewing-architecture-principles/
│   ├── reviewing-bugs/
│   ├── reviewing-compliance/
│   ├── reviewing-documentation/     # Has 2 reference files
│   ├── reviewing-performance/
│   ├── reviewing-security/          # Expanded example (all 7 review skills follow this pattern):
│   │   ├── SKILL.md                 # Core skill instructions (~30-50 lines)
│   │   └── references/              # Detailed patterns (loaded on-demand)
│   │       └── common-vulnerabilities.md
│   ├── reviewing-technical-debt/
│   └── synthesis-instructions/      # Internal: synthesis process, output schema (loaded by synthesis agents via skills field)
│       └── SKILL.md
├── languages/                       # Language-specific configs
│   ├── dotnet.md                    # .NET/C# checks
│   ├── nodejs.md                    # Node.js/TypeScript checks
│   └── react.md                     # React checks (extends Node.js)
├── shared/
│   ├── review-orchestration-code.md # Code review: phases, invocation patterns, gaps mode behavior
│   ├── review-orchestration-docs.md # Docs review: phases, invocation patterns, gaps mode behavior
│   ├── review-validation-code.md    # Code validation: batch validation, aggregation, auto-validation patterns
│   ├── review-validation-docs.md    # Docs validation: batch validation, aggregation, auto-validation patterns
│   ├── skill-handling.md            # Skill resolution and orchestration (loaded when --skills used)
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
model: <opus|sonnet>           # thorough-mode default; gaps always Sonnet
tools: ["Read", "Grep", "Glob"]
skills: ["code-review:agent-review-instructions"]  # All 15 non-synthesis agents
maxTurns: 5                    # Limits agent execution turns
permissionMode: dontAsk        # Auto-deny permissions (read-only agents)
---
```

The `skills` field loads static agent configuration at agent startup. Non-synthesis agents load `code-review:agent-review-instructions` (MODE definitions, false positive rules, output schema). Synthesis agents load `code-review:synthesis-instructions` (input format, review process, output schema, guidelines).

Other Plugin Reference fields (`disallowedTools`, `mcpServers`, `hooks`, `memory`) are not used.

**Color rules:** Minimize conflicts within each parallel phase; reuse across sequential phases is fine. Use `white` for overflow. Do not change existing colors without necessity.

**Agent body structure (two formats):**

- **Opus agents** (architecture, bug-detection, performance, security, technical-debt, accuracy, completeness, examples): `## MODE Checklists` → `## Output`. No `## Review Process` or `### Step N:` headings — Opus needs domain context, not analysis methodology.
- **Sonnet agents** (api-contracts, compliance, error-handling, test-coverage, clarity, consistency, structure): `## Review Process` → `### Step 1-N:` methodology → `## Output`. Retains analysis steps — Sonnet benefits from explicit guidance.
- **Synthesis agents** (synthesis-code, synthesis-docs): Domain-specific body only (Category Key Mapping, Step 2 interaction patterns, example YAML). Shared process loaded via `code-review:synthesis-instructions` skill.

**MODE labels:** `**thorough:**`, `**gaps:**`, `**quick:**` (no suffixes like "mode - Focus on:").

**Output section:** All 15 non-synthesis agents use `## Output` with: category, description guidance, severity thresholds (compressed single-line `Thresholds: Critical=...; Major=...`), and category-specific YAML extra fields. No "See Output Schema in additional_instructions" — that content is injected by the orchestrator at runtime.

### Deep Review Pipeline

1. **Phase 1** (9 agents parallel): Thorough mode review (5 Opus, 4 Sonnet)
2. **Phase 2** (5 Sonnet agents parallel): Gaps mode with Phase 1 findings as context
3. **Synthesis** (up to 5 agents parallel): Cross-cutting concern detection (requires findings in BOTH input categories; single-category insights are rejected)
4. **Validation**: All issues validated before output

### Agent Content Distribution

Static agent instructions are self-loaded via the `skills` field: non-synthesis agents load `code-review:agent-review-instructions` (MODE, false positive rules, output schema); synthesis agents load `code-review:synthesis-instructions` (input format, review process, output schema, guidelines). The orchestrator distributes only dynamic content via `additional_instructions`: language-specific checks from `languages/*.md` and LSP diagnostic codes (when LSP is available). Category-specific false positive rules remain in each agent's `## False Positives` section.

Validation, auto-validation, and output format content is in separate files (`review-validation-code.md`, `review-validation-docs.md`), loaded on-demand at Steps 9-12 (post-review). This progressive disclosure keeps validation and output format content out of the Opus context during the expensive Steps 7-8 phase.

See `shared/review-orchestration-code.md` and `shared/review-orchestration-docs.md` for orchestration details.

### Gaps Mode Agent Selection Rationale

**Selected agents:** bug-detection, compliance, performance, security, technical-debt

**Rationale:**
1. **High complexity domains**: Security, performance, and bugs have many subtle edge cases that benefit from a second pass with fresh perspective
2. **Domain overlap potential**: Compliance gaps catches subtle rule misinterpretations and edge cases the thorough pass accepted too readily. Technical-debt gaps catches context-dependent debt requiring cross-file analysis that single-pass thorough misses.
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

Settings loading logic is inlined in each orchestration skill (Step 2). See `README.md` for full documentation.

### Language Detection

- **Node.js/TypeScript**: Detected by `package.json`
- **React**: Detected by `react` or `react-dom` in package.json dependencies (extends Node.js checks)
- **.NET/C#**: Detected by `*.csproj`, `*.sln`, or `*.slnx`

All category anchors referenced in `review-orchestration-code.md` (`{#architecture}`, `{#bugs}`, `{#errors}`, `{#performance}`, `{#security}`, `{#debt}`, `{#tests}`) must have content in language files. React inherits Node.js anchors; add React-specific overrides only.

### Skill Structure (Progressive Disclosure)

Each skill follows progressive disclosure: `SKILL.md` is always loaded when triggered; `references/` subdirectories are loaded on-demand (one level deep from SKILL.md). Skills are self-contained with their own workflow procedures. Review skills provide unique value (scope prioritization, FP adjustments, reference files, methodology) but do not duplicate agent category checklists — categories are in agent files only. Two internal skills (not user-facing) provide static configuration loaded at startup via the `skills` frontmatter field: `agent-review-instructions` (MODE, FP rules, output schema for non-synthesis agents) and `synthesis-instructions` (input format, review process, output schema for synthesis agents).

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
- **Agent model assignments**: Edit agent YAML frontmatter (`model` field)
- **Code review orchestration**: Edit `shared/review-orchestration-code.md` (phases, invocation patterns, language-specific focus, gaps mode behavior)
- **Docs review orchestration**: Edit `shared/review-orchestration-docs.md` (phases, invocation patterns, gaps mode behavior)
- **Agent common instructions**: Edit `skills/agent-review-instructions/SKILL.md` (MODE, FP rules, output schema — loaded by non-synthesis agents via `skills` field)
- **Synthesis common instructions**: Edit `skills/synthesis-instructions/SKILL.md` (input format, review process, output schema — loaded by synthesis agents via `skills` field)

### Validation & Output
- **Validation rules (code)**: Edit `shared/review-validation-code.md` (validation process, aggregation, auto-validation patterns)
- **Validation rules (docs)**: Edit `shared/review-validation-docs.md` (validation process, aggregation, auto-validation patterns)
- **Output format/generation**: Edit the "Output Format" section in `shared/review-validation-code.md` and `shared/review-validation-docs.md`
- **Severity definitions**: Each agent defines calibrated thresholds in its own file under `agents/code/` or `agents/docs/`

### Skills & Language
- **Skills**: Edit `skills/*/SKILL.md`; add patterns to `references/`
- **Language-specific checks**: Edit files in `languages/`

### Orchestration Skills & Settings
- **Orchestration skill arguments**: Edit skill YAML frontmatter in `skills/code-review/SKILL.md` or `skills/docs-review/SKILL.md`
- **Settings options**: Edit Step 2 in `skills/code-review/SKILL.md` and `skills/docs-review/SKILL.md`, and `templates/code-review.local.md.example`
- **Pre-existing issue detection**: Edit "Pre-Existing Issue Detection" in `skills/code-review/SKILL.md` (Steps 3 & 5, staged section)

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

Model-aware compression:
- **Opus agents**: No analysis steps (Step 2+). `## MODE Checklists` with high-level triggers (1-3 lines for thorough). `## Output` merges description + thresholds + YAML fields. "Claude is already smart" — only add domain context Claude doesn't already have. Thorough items must NOT repeat the agent's `description` field — the description defines the domain; thorough adds only calibration thresholds, specific scope steering, or scope the description doesn't cover. If thorough would merely restate the description, omit it.
- **Sonnet agents**: Keep `## Review Process` with methodology steps. `## Output` merges description + thresholds + YAML fields (no separate Report step or Output Schema section).
- **All agents**: Preserve gaps/quick mode items, severity thresholds, category-specific YAML fields verbatim. MODE differentiation (thorough/gaps/quick sections) intact. MODE labels: `**thorough:**` not `**thorough mode - Focus on:**`.

### Content Audience

**CRITICAL:** Nearly all content in this repository is written for LLM consumption. Do not adjust content for human readability unless the file is explicitly listed as human-readable below.

**LLM-consumed content** (everything except the files listed below):

| Subcategory | Files | Consumer | When |
|-------------|-------|----------|------|
| Development-time | `CLAUDE.md` | Claude Code | When authoring/modifying the plugin |
| Runtime | `agents/`, `shared/`, `skills/`, `languages/`, `templates/` | Opus orchestrator + agents | During every review execution |

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
- Tables with agent/skill names
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
- Orchestration skill workflows in `skills/code-review/SKILL.md` and `skills/docs-review/SKILL.md`
- Sonnet agent workflows in `agents/code/*.md` and `agents/docs/*.md` (Opus agents have no step headings)
- Other skill workflows in `skills/*/SKILL.md`

**Rationale:** Consistency across orchestration skill and Sonnet agent workflows.

**Step layout in orchestration skills:**
- Steps 1, 2, 4, 6: Pre-review setup (methodology, settings, context, skills) - inlined in each skill
- Steps 3, 5: Input validation, content gathering - skill-specific
- Steps 7-8: Review execution, synthesis - references to orchestration files
- Steps 9-12: Post-review (validation, aggregation, output, write) - compressed inline

### Orchestration Skill Step Inlining

Each orchestration skill inlines its pre-review setup steps (Steps 1-6) directly, including settings loading and context discovery (formerly `pre-review-setup.md`). The code review skill also inlines staged processing (formerly `staged-processing.md`). Steps 7-8 reference orchestration files for review execution and synthesis. Steps 9-12 are compressed inline.

**Rationale:** A shared `skill-common-steps.md` was tried but created a second level of indirection (skills → common-steps → shared files), adding an extra file to the Opus context window. Inlining keeps orchestration skills self-contained with one-hop references to shared files.

**Revisit if:** orchestration skill count exceeds 4, shared steps diverge significantly, or a way is found to share steps without extra Opus context files.

### Intentional Cross-File Duplication

| File Pair | Shared Content | ~Lines | Rationale |
|-----------|---------------|--------|-----------|
| `skills/code-review/SKILL.md` / `skills/docs-review/SKILL.md` | Settings loading, Context Discovery | ~35 | Only one skill runs per execution; inlined from former `pre-review-setup.md` |
| `review-orchestration-code.md` / `review-orchestration-docs.md` | File Entry Schema, Gaps Mode core | ~50 | Only one loaded per execution; extracting adds a file read |
| `review-validation-code.md` / `review-validation-docs.md` | Batch Validation, Validator Schema, Common FP, Verdicts, Aggregation, Output Format | ~110 | Same rationale: only one loaded per execution |

Synthesis agents (`synthesis-code-agent.md` / `synthesis-docs-agent.md`) formerly duplicated ~50 lines of shared process content. Now extracted to `skills/synthesis-instructions/SKILL.md`; agents retain only domain-specific content (category mappings, interaction patterns, examples).

**Maintenance rule:** When modifying shared content in one file, `grep -r` for the same section heading in the paired file and update both.

### Orchestration Skills (code-review, docs-review)

Review orchestration was migrated from `commands/` to `skills/` per Anthropic's Plugin Reference recommendation. The skill YAML schema supports all needed fields: `name`, `allowed-tools`, `description`, `argument-hint`, `model`. Both orchestration skills use `model: opus`. The `allowed-tools` list includes `Task` (agent invocation), git-scoped `Bash` patterns, `Read`, `Write`, `Glob`.

**CRITICAL:** These orchestration skills must NOT use `context: fork`. Forked skills run in a subagent that cannot spawn other subagents. The review orchestration dispatches agents via the `Task` tool, which requires running in the main thread.

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

## Common Gotchas

- **Trailing newlines**: Subagent file rewrites often drop trailing newlines. Verify with `tail -c 1` after bulk operations.
- **Reference integrity**: When moving content between files, `grep -r "section name"` before AND after to catch all references (commands, skills, shared files).
- **Content category violations**: Before adding content to `shared/` or `shared/references/`, ask "who consumes this?" If the answer is "Claude Code when modifying the plugin," it belongs in CLAUDE.md, not runtime directories.
- **Synthesis constraint**: Synthesis insights require findings in BOTH input categories. Single-category insights are rejected at validation.
- **Validation progressive disclosure**: Orchestration files (`review-orchestration-*.md`) cross-reference validation files (`review-validation-*.md`). Do not re-merge them — the split reduces Opus context by ~53% during Steps 7-8.

### Auto-Validation Pattern Design Notes

These notes apply when modifying patterns in `review-validation-{code|docs}.md`:
- Patterns use hybrid list format: self-evident patterns (N/A regex or obvious behavior) omit regex; calibration-critical patterns (behavioral qualifiers, variable name lists, non-obvious matching rules) include inline regex after em dash
- Patterns are case-insensitive for SQL keywords
- Patterns require at least one character (`[^'"]+`) to avoid matching empty string placeholders like `password = ""`
- Patterns check for common variable names indicating user input: `req`, `request`, `params`, `query`, `body`, `input`, `user`
- Empty catch pattern allows a single comment line to avoid flagging intentional empty catches with explanation
- Language-labeled patterns use `[Node.js]`, `[React]`, or `[.NET]` tags after the severity tag. Unlabeled patterns apply to all languages. The orchestrator filters language-labeled patterns by detected language at validation time.

### Content Strategy Rationale

Agents have access to Read, Grep, and Glob tools. For Phase 2 (gaps) and Synthesis, providing file paths instead of full content allows agents to fetch content on-demand, reducing baseline token usage while preserving capability. Phase 1 provides full file content and diffs; Phase 2 provides diffs only; Synthesis provides file paths only. The operational details are in the Review Sequence sections of `review-orchestration-code.md`.

## Version Management

**Version bump rules:** Patch = fixes/docs. Minor = new features/agents/commands. Major = breaking changes.

**Version locations** (all three must match): `plugin.json`, `marketplace.json` (repo root), `README.md` ("Current Version"). Per Anthropic guidance, only `plugin.json` is authoritative — individual agent/skill files must NOT have version fields.

**Release steps:**
1. Find previous tag and diff changes: `git tag -l --sort=-v:refname | head -5` then `git log <prev>..HEAD --oneline -- claude-code/plugins/code-review/`
2. Update CHANGELOG.md first (Keep a Changelog format, base on `git diff` not commit messages)
3. Update version in all 3 locations
4. Verify: `grep -r "<prev>" --include="*.md" --include="*.json" | grep -v CHANGELOG` (expect 0) and `grep -r "<new>" ... | wc -l` (expect 3)
5. Commit, tag, push: `git tag v<new> && git push origin main && git push origin v<new>`
