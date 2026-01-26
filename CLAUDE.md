# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Claude Code plugin repository containing the **Code Review Plugin** (v3.1.4) - a modular 10-agent architecture with:
- Two-phase sequential review (thorough → gaps with context passing)
- Cross-agent synthesis for ripple effect detection
- Actionable fix outputs (inline diffs and Claude Code prompts)
- Support for Node.js/TypeScript and .NET/C# projects

## Plugin Commands

| Command | Description |
|---------|-------------|
| `/deep-review <file1> [file2...] [--output-file <path>]` | Deep review: Phase 1 (9 agents) → Phase 2 (5 gaps agents) → Synthesis (4 agents) |
| `/deep-review-staged [--output-file <path>]` | Deep review of staged git changes with full pipeline |
| `/quick-review <file1> [file2...] [--output-file <path>]` | Quick review (4 agents + 3 synthesis agents) |
| `/quick-review-staged [--output-file <path>]` | Quick review of staged git changes (7 agent invocations) |

**Note:** All review commands also accept:
- `--language nodejs|dotnet` to force language detection
- `--prompt "<instructions>"` to pass additional instructions to agents
- `--skills <skill1,skill2,...>` to enable skill-informed orchestration (orchestrator interprets skills and generates tailored `skill_instructions` per agent)

## Plugin Skills

| Skill | Trigger Phrases |
|-------|-----------------|
| `security-review` | "security review", "check for vulnerabilities", "audit security" |
| `performance-review` | "check performance", "find slow code", "optimize" |
| `bug-review` | "find bugs", "check for errors", "find edge cases" |
| `compliance-review` | "check CLAUDE.md compliance", "review against standards" |
| `technical-debt-review` | "find technical debt", "check for deprecated code", "identify dead code" |

## Architecture

### Directory Structure

```
claude-code/plugins/code-review/
├── .claude-plugin/plugin.json       # Plugin metadata (v3.1.4)
├── commands/                        # Thin orchestration documents (reference shared/)
│   ├── deep-review.md               # Deep file review (18 agent invocations)
│   ├── deep-review-staged.md        # Deep staged review (18 agent invocations)
│   ├── quick-review.md              # Quick file review (7 invocations)
│   └── quick-review-staged.md       # Quick staged review (7 invocations)
├── agents/                          # Modular agent definitions (10 agents)
│   ├── compliance-agent.md          # AI instructions compliance
│   ├── bug-detection-agent.md       # Logical errors & edge cases
│   ├── security-agent.md            # Security vulnerabilities
│   ├── performance-agent.md         # Performance issues
│   ├── architecture-agent.md        # Architecture patterns
│   ├── api-contracts-agent.md       # API compatibility
│   ├── error-handling-agent.md      # Error handling gaps
│   ├── test-coverage-agent.md       # Test coverage gaps
│   ├── technical-debt-agent.md      # Technical debt detection
│   └── synthesis-agent.md           # Cross-agent insights
├── skills/                          # Targeted review skills (progressive disclosure)
│   ├── security-review/
│   │   ├── SKILL.md                 # Core skill instructions
│   │   ├── references/              # Detailed patterns (loaded on-demand)
│   │   │   └── common-vulnerabilities.md
│   │   └── examples/                # Sample output format
│   │       └── example-output.md
│   ├── performance-review/
│   │   ├── SKILL.md
│   │   ├── references/
│   │   │   └── performance-patterns.md
│   │   └── examples/
│   │       └── example-output.md
│   ├── bug-review/
│   │   ├── SKILL.md
│   │   ├── references/
│   │   │   └── common-bugs.md
│   │   └── examples/
│   │       └── example-output.md
│   ├── compliance-review/
│   │   ├── SKILL.md
│   │   ├── references/
│   │   │   └── compliance-patterns.md
│   │   └── examples/
│   │       └── example-output.md
│   └── technical-debt-review/
│       ├── SKILL.md
│       ├── references/
│       │   └── technical-debt-patterns.md
│       └── examples/
│           └── example-output.md
├── languages/                       # Language-specific configs
│   ├── nodejs.md                    # Node.js/TypeScript checks
│   └── dotnet.md                    # .NET/C# checks
├── shared/
│   ├── orchestration-sequence.md    # Phase definitions and model selection (authoritative)
│   ├── agent-invocation-pattern.md  # Task invocation pattern for agents
│   ├── agent-common-instructions.md # Common MODE, false positives, gaps, output schema
│   ├── command-common-steps.md      # Common workflow steps for all commands
│   ├── settings-loader.md           # Settings loading and application
│   ├── input-validation-files.md    # File-based input validation
│   ├── input-validation-staged.md   # Staged input validation
│   ├── content-gathering-files.md   # File-based content gathering
│   ├── content-gathering-staged.md  # Staged content gathering
│   ├── context-discovery.md         # Context discovery instructions
│   ├── review-workflow.md           # Workflow steps and settings application
│   ├── skill-orchestration.md       # Skill-informed orchestration (loaded when --skills used)
│   ├── skill-resolver.md            # Skill resolution and structured parsing
│   ├── skill-common-workflow.md     # Common skill workflow steps (lean, references details)
│   ├── validation-rules.md          # Validation process
│   ├── output-format.md             # Output templates (with fix_type)
│   ├── output-generation.md         # Output generation and file writing
│   ├── output-schema-base.md        # Base YAML schema for all agents
│   ├── severity-definitions.md      # Severity classification
│   ├── gaps-mode-rules.md           # Rules for gaps mode operation
│   ├── false-positives.md           # False positive rules
│   └── references/                  # Detailed reference content (progressive disclosure)
│       ├── scope-determination.md   # Detailed scope options and edge cases
│       ├── validation-details.md    # Batch validation and false positive handling
│       └── skill-troubleshooting.md # Common issues and solutions
├── templates/
│   └── code-review.local.md.example # Settings template for users
└── README.md
```

### Agent Configuration

See the following files for authoritative agent configuration:
- `shared/orchestration-sequence.md` - Model selection table, phase definitions
- `shared/agent-invocation-pattern.md` - Task invocation template
- `shared/agent-common-instructions.md` - Common MODE, false positives, gaps behavior
- `shared/review-workflow.md` - Usage tracking protocol, settings application

### Agent Colors

Each agent has a unique color for visual identification during parallel execution:

| Agent | Color |
|-------|-------|
| compliance-agent | blue |
| bug-detection-agent | red |
| security-agent | magenta |
| performance-agent | yellow |
| architecture-agent | cyan |
| api-contracts-agent | green |
| error-handling-agent | orange |
| test-coverage-agent | purple |
| technical-debt-agent | brown |
| synthesis-agent | white |

**Color Usage Notes:**
- All 9 review agents have unique colors within Phase 1
- Phase 2 reuses colors from Phase 1 (compliance, bug, security, performance, technical-debt) but runs sequentially after Phase 1 completes
- Synthesis phase runs 4 parallel instances of synthesis-agent, all using white (same agent type)
- Color conflicts within a phase are avoided; color reuse across sequential phases is acceptable

### Deep Review Pipeline

1. **Phase 1** (9 agents parallel): Thorough mode review (4 Opus, 5 Sonnet)
2. **Phase 2** (5 Sonnet agents parallel): Gaps mode with Phase 1 findings as context
3. **Phase 3** (4 synthesis agents parallel): Cross-cutting concern detection
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
- `shared/agent-common-instructions.md` - Common agent instructions (MODE, false positives, gaps, output schema)
- `shared/command-common-steps.md` - Common workflow steps shared by all commands
- `shared/settings-loader.md` - Settings loading and application
- `shared/input-validation-*.md` - Input validation for file/staged commands
- `shared/content-gathering-*.md` - Content gathering for file/staged commands
- `shared/context-discovery.md` - AI Agent Instructions and project type detection
- `shared/review-workflow.md` - Workflow steps and settings application
- `shared/skill-orchestration.md` - Skill-informed orchestration (lazy-loaded when --skills used)
- `shared/skill-resolver.md` - Skill resolution and structured parsing for orchestrator interpretation
- `shared/skill-common-workflow.md` - Lean workflow steps for skills (references `shared/references/` for details)
- `shared/validation-rules.md` - Issue validation process
- `shared/output-schema-base.md` - Base YAML schema all agents must use
- `shared/output-format.md` - Output formatting templates with fix_type
- `shared/output-generation.md` - Output generation and file writing process
- `shared/severity-definitions.md` - Severity classification (Critical, Major, Minor, Suggestion)
- `shared/gaps-mode-rules.md` - Rules for gaps mode (deduplication, focus areas)
- `shared/false-positives.md` - False positive rules to prevent invalid findings
- `shared/usage-tracking.md` - Usage tracking schema and protocol
- `skills/*/SKILL.md` - Skill definitions (reference skill-common-workflow.md)
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
custom_rules: []
---

# Project-specific instructions for review agents
```

See `shared/settings-loader.md` for loading logic and `README.md` for full documentation.

## Language Detection

- **Node.js/TypeScript**: Detected by `package.json`
- **.NET/C#**: Detected by `*.csproj`, `*.sln`, or `*.slnx`

## Making Changes

When modifying the plugin:

1. **Agent behavior**: Edit agent files in `agents/` directory
2. **Language-specific checks**: Edit files in `languages/` directory
3. **Validation rules**: Edit `shared/validation-rules.md`
4. **Output format**: Edit `shared/output-format.md`
5. **Workflow steps**: Edit `shared/review-workflow.md`
6. **Orchestration sequence**: Edit `shared/orchestration-sequence.md` (phase definitions, model selection)
7. **Agent invocation pattern**: Edit `shared/agent-invocation-pattern.md`
8. **Common agent instructions**: Edit `shared/agent-common-instructions.md` (MODE, false positives, gaps)
9. **Common command steps**: Edit `shared/command-common-steps.md` (settings, context, validation, output)
10. **Common skill steps**: Edit `shared/skill-common-workflow.md`
11. **Command arguments**: Edit command YAML frontmatter in `commands/`
12. **Skills**: Edit skill files in `skills/*/SKILL.md`
13. **Skill references**: Add detailed patterns to `skills/*/references/`
14. **Skill examples**: Add sample outputs to `skills/*/examples/`
15. **Settings options**: Edit `shared/settings-loader.md` and `templates/code-review.local.md.example`

## Skill Structure (Progressive Disclosure)

Each skill follows progressive disclosure pattern:

1. **SKILL.md** (~130-170 lines): Core workflow loaded when skill triggers
2. **shared/skill-common-workflow.md**: Common workflow steps (scope, context, validation, reporting)
3. **references/** (optional): Detailed patterns loaded on-demand
4. **examples/** (optional): Sample output formats for reference

```
skills/skill-name/
├── SKILL.md              # Always loaded when skill triggers (~130-170 lines)
├── references/           # Loaded when Claude needs detailed patterns
│   └── patterns.md
└── examples/             # Loaded when Claude needs output format reference
    └── example-output.md
```

Skills reference `${CLAUDE_PLUGIN_ROOT}/shared/skill-common-workflow.md` for common procedures (determining scope, gathering context, validating findings, and reporting results) to avoid duplication across skills.

## Skill-Informed Orchestration

When `--skills` is provided to review commands, the orchestrator (running as Opus) interprets skill content and makes orchestration decisions:

1. **Skill Resolution**: Skills are resolved to SKILL.md files via `shared/skill-resolver.md`
2. **Structured Parsing**: Skills are parsed into structured data (focus_categories, auto_validated_patterns, false_positive_rules, methodology)
3. **Agent-Specific Instructions**: The orchestrator generates tailored `skill_instructions` per agent:
   - Review skills (security-review, etc.) target their primary agent with checklists and focus areas
   - Methodology skills (superpowers:brainstorming, etc.) apply universally to all agents
4. **Validation Adjustments**: Auto-validated patterns skip validation; false positive rules filter findings
5. **Synthesis Adjustments**: Skill-specific cross-cutting questions may be added

See `shared/skill-orchestration.md` for implementation details.

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

   **Required:**
   - `claude-code/plugins/code-review/.claude-plugin/plugin.json`
   - `.claude-plugin/marketplace.json` (repository root)
   - `claude-code/plugins/code-review/README.md` (Current Version line)
   - `CLAUDE.md` version references (this file)

   **Recommended:**
   - All agent files: `agents/*.md` (10 files)
   - All skill files: `skills/*/SKILL.md` (5 files)

5. **Verify versions are consistent:**
   ```bash
   # Verify no old version remains (exclude CHANGELOG history)
   grep -r "<prev>" --include="*.md" --include="*.json" | grep -v CHANGELOG

   # Verify new version count (~19: 1 plugin.json + 1 marketplace.json + 1 README + 1 CLAUDE.md + 10 agents + 5 skills)
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
