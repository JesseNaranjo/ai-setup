# Code Review Plugin

Modular 10-agent code review with parameterized modes for Node.js and .NET projects. Also includes documentation review commands with 6 specialized documentation agents.

## Overview

The Code Review Plugin provides automated, in-depth code review using 10 specialized agents that analyze code from different perspectives. Each agent supports multiple review modes (thorough, gaps, quick) for flexible review depth.

Additionally, the plugin provides documentation review commands that analyze project documentation for accuracy, clarity, completeness, consistency, examples quality, and structure.

### Key Features

- **Modular 10-Agent Architecture**: Each agent is a separate file in `agents/` for easy customization
- **Parameterized Modes**: thorough, gaps, and quick modes for different review depths
- **Language-Aware**: Specialized checks for Node.js/TypeScript and .NET/C# (configs in `languages/`)
- **Targeted Skills**: Security, performance, bug, compliance, and technical debt review skills
- **Validation Layer**: Every issue is independently validated to reduce false positives
- **Severity Classification**: Issues categorized as Critical, Major, Minor, or Suggestion
- **Consensus Scoring**: Issues flagged by multiple agents get higher confidence

## Commands

### `/deep-code-review`

Comprehensive code review using all 9 review agents plus synthesis (19 invocations) with thorough + gaps modes for maximum coverage.

```bash
/deep-code-review <file1> [file2...] [--output-file <path>] [--language <nodejs|dotnet>] [--prompt "<instructions>"] [--skills <skills>]
```

### `/deep-code-review-staged`

Comprehensive code review of staged git changes using all 9 review agents plus synthesis (19 invocations) with thorough + gaps modes.

```bash
/deep-code-review-staged [--output-file <path>] [--language <nodejs|dotnet>] [--prompt "<instructions>"] [--skills <skills>]
```

### `/quick-code-review`

Fast review using 4 agents (7 invocations) focusing on critical issues (bugs, security, errors, tests).

```bash
/quick-code-review <file1> [file2...] [--output-file <path>] [--language <nodejs|dotnet>] [--prompt "<instructions>"] [--skills <skills>]
```

### `/quick-code-review-staged`

Fast review of staged git changes using 4 agents (7 invocations) focusing on critical issues (bugs, security, errors, tests).

```bash
/quick-code-review-staged [--output-file <path>] [--language <nodejs|dotnet>] [--prompt "<instructions>"] [--skills <skills>]
```

## Documentation Review Commands

### `/deep-docs-review`

Comprehensive documentation review using all 6 documentation agents (13 invocations) with thorough + gaps modes.

```bash
/deep-docs-review [file1...] [--output-file <path>] [--prompt "<instructions>"]
```

If no files are specified, discovers and reviews all documentation files (README.md, CLAUDE.md, docs/*, etc.).

### `/quick-docs-review`

Fast documentation review using 4 agents (7 invocations) focusing on critical issues (accuracy, clarity, examples, structure).

```bash
/quick-docs-review [file1...] [--output-file <path>] [--prompt "<instructions>"]
```

### Documentation Categories

| Category | Focus Areas |
|----------|-------------|
| **Accuracy** | Code-doc sync, API signatures, version accuracy |
| **Clarity** | Readability, jargon, audience appropriateness |
| **Completeness** | Missing sections, undocumented features |
| **Consistency** | Terminology, formatting, voice uniformity |
| **Examples** | Code example validity, completeness |
| **Structure** | Organization, links, AI instruction standardization |

### AI Instruction File Standardization

Documentation reviews include standardization checks for AI agent instruction files:

- `/.ai/AI-AGENT-INSTRUCTIONS.md` - Comprehensive coding standards (required location)
- `/CLAUDE.md` - Claude Code quick reference (must reference .ai/)
- `/.github/copilot-instructions.md` - GitHub Copilot quick reference

## Configuration

Customize the plugin's behavior per-project by creating a settings file.

### Quick Start

1. Copy the template to your project:
   ```bash
   mkdir -p .claude
   cp ~/.claude/plugins/*/code-review/templates/code-review.local.md.example .claude/code-review.local.md
   ```

2. Edit `.claude/code-review.local.md` to customize settings

3. Add to `.gitignore`:
   ```gitignore
   .claude/*.local.md
   ```

> **Note:** Settings are read each time a review command runs. No restart required.

### Settings File Format

Create `.claude/code-review.local.md` in your project root:

```markdown
---
# Enable/disable the plugin
enabled: true

# Default output directory (default: "." = project root)
output_dir: "."

# Agents to skip (options: compliance, bugs, security, performance, architecture,
#                         api-contracts, error-handling, test-coverage, technical-debt)
skip_agents: []

# Minimum severity to report (options: critical, major, minor, suggestion)
min_severity: "suggestion"

# Language override (options: nodejs, dotnet, or empty for auto-detect)
language: ""
---

# Project-Specific Instructions

Add context for review agents here. This content is passed to all agents.
```

### Configuration Options

| Setting | Default | Description |
|---------|---------|-------------|
| `enabled` | `true` | Enable/disable plugin for this project |
| `output_dir` | `"."` | Directory for review output files |
| `skip_agents` | `[]` | List of agents to exclude from reviews |
| `min_severity` | `"suggestion"` | Filter output to this severity or higher |
| `language` | `""` | Force language (overridden by `--language` flag) |
| `additional_test_patterns` | `[]` | Extra glob patterns for test files |

### Example Configurations

**Skip architecture reviews and only show major issues:**
```yaml
---
skip_agents: ["architecture", "api-contracts"]
min_severity: "major"
---
```

**Save reviews to a dedicated folder:**
```yaml
---
output_dir: "./docs/reviews"
---
```

**Provide project context:**
```yaml
---
enabled: true
---

# Project Context

This is a financial application. Pay extra attention to:
- Input validation on all endpoints
- SQL injection prevention
- Error messages that don't leak sensitive data
```

### Priority Order

Command-line flags extend or override settings file:
1. `--output-file` overrides `output_dir`
2. `--language` overrides `language`
3. `--prompt` appends to project instructions (does not replace)

### Additional Prompt Instructions

Use `--prompt` to pass ad-hoc instructions to all review agents without editing your settings file. This is useful for:
- Focusing reviews on specific concerns
- Providing one-off context for a particular review
- Experimenting with different review approaches

#### Example: Brainstorming-Style Deep Exploration

```bash
/deep-code-review src/auth/*.ts --prompt "Before flagging issues, brainstorm multiple attack vectors and failure modes for each function. Consider: What assumptions does this code make? What happens if those assumptions are violated? What edge cases might the original developer have missed? Explore creatively before concluding."
```

#### Example: Systematic Checklist Approach

```bash
/quick-code-review-staged --prompt "For each file, systematically check: 1) Input validation gaps, 2) Error handling completeness, 3) Resource cleanup, 4) Concurrency issues, 5) Security boundaries. Don't skip any category even if it seems unlikely."
```

#### Example: Domain-Specific Focus

```bash
/deep-code-review src/api/payments.ts --prompt "This is a payment processing module. Focus on: financial calculation precision, transaction atomicity, audit trail completeness, PCI compliance patterns. Flag anything that could lead to money loss or compliance violations."
```

#### Example: Threat Model Guidance

```bash
/deep-code-review src/api/*.ts --prompt "Assume an attacker has valid credentials but is trying to access other users' data. Focus on authorization checks, IDOR vulnerabilities, and data leakage in error messages."
```

#### Combining with Project Settings

For persistent instructions, use the markdown body in `.claude/code-review.local.md`:

```yaml
---
enabled: true
---

# Review Philosophy

Apply brainstorming principles: explore multiple interpretations of each code path before concluding. Ask "what if?" for each assumption.

Use systematic analysis: check each security category even when code appears safe.
```

Then use `--prompt` for one-off additions:

```bash
/deep-code-review src/auth/*.ts --prompt "Additionally, this PR introduces OAuth. Verify token handling follows OWASP guidelines."
```

#### Effective Prompt Strategies

| Goal | Approach |
|------|----------|
| Thorough exploration | Describe the *mindset* you want agents to adopt |
| Specific focus areas | List concrete categories or concerns to check |
| Domain context | Explain what the code does and why it matters |
| Known risks | Point agents toward specific threat models |
| Review philosophy | Describe how to approach analysis (systematic, creative, etc.) |

### Embedding Skills in Reviews

Use `--skills` to enhance reviews with skill-specific knowledge and methodologies. The orchestrator interprets skill content and generates tailored instructions for each agent.

#### Syntax

```bash
--skills <skill1,skill2,...>
```

#### Skill References

| Format | Example | Description |
|--------|---------|-------------|
| Plugin-local | `security-review` | Skills within the code-review plugin |
| External plugin | `superpowers:brainstorming` | Skills from other plugins (plugin:skill) |

#### Examples

**Embed brainstorming methodology:**
```bash
/deep-code-review src/auth/*.ts --skills superpowers:brainstorming
```
All agents will explore multiple interpretations and failure modes before flagging issues.

**Combine multiple skills:**
```bash
/deep-code-review src/api/*.ts --skills security-review,superpowers:systematic-debugging
```
Security agent receives targeted security checklists; all agents receive debugging methodology.

**Use with --prompt:**
```bash
/deep-code-review src/payments.ts --skills superpowers:brainstorming --prompt "Focus on financial calculation precision"
```
Combines skill methodology with specific instructions.

#### How It Works

1. Skill names are resolved to SKILL.md files
2. Skills are parsed into structured data (categories, patterns, rules, methodology)
3. The orchestrator generates tailored `skill_instructions` per agent:
   - **Review skills** (security-review, etc.) → Targeted to their primary agent with focus areas and checklists
   - **Methodology skills** (brainstorming, etc.) → Applied universally to all agents
4. Auto-validated patterns skip the validation step for high-confidence findings
5. False positive rules are applied across all agents

#### Benefits of Skill-Informed Orchestration

| Benefit | Description |
|---------|-------------|
| **Agent-specific tailoring** | Security skills target security-agent; methodology applies to all |
| **Smarter validation** | Auto-validated patterns reduce unnecessary validation overhead |
| **Reduced false positives** | Skill-defined rules filter out known false positive patterns |
| **Skill-informed synthesis** | Cross-cutting analysis considers skill-specific concerns |

#### Available Plugin Skills

| Skill | Primary Agent | Focus |
|-------|---------------|-------|
| `architecture-principles-review` | architecture-agent | SOLID, DRY, YAGNI violations |
| `bug-review` | bug-detection-agent | Logical errors and edge cases |
| `compliance-review` | compliance-agent | CLAUDE.md and coding standards |
| `performance-review` | performance-agent | Performance issues and optimization |
| `security-review` | security-agent | Security vulnerabilities and OWASP patterns |
| `technical-debt-review` | technical-debt-agent | Deprecated code, outdated patterns, dead code |

## Skills

Targeted review skills for specific concerns:

| Skill | Trigger Phrases |
|-------|-----------------|
| `architecture-principles-review` | "check SOLID", "find DRY violations", "check YAGNI", "architecture principles" |
| `bug-review` | "find bugs", "check for errors", "find edge cases" |
| `compliance-review` | "check CLAUDE.md compliance", "review against standards" |
| `docs-review` | "review documentation", "check docs", "audit README", "verify AI instructions" |
| `performance-review` | "check performance", "find slow code", "optimize" |
| `security-review` | "security review", "check for vulnerabilities", "audit security" |
| `technical-debt-review` | "find technical debt", "check for deprecated code", "identify dead code" |

## Architecture

### Directory Structure

```
code-review/
├── .claude-plugin/
│   └── plugin.json              # Plugin metadata (v3.3.0)
├── commands/
│   ├── deep-docs-review.md      # Deep documentation review (13 invocations)
│   ├── deep-code-review.md      # Deep file review (19 invocations)
│   ├── deep-code-review-staged.md    # Deep staged review (19 invocations)
│   ├── quick-docs-review.md     # Quick documentation review (7 invocations)
│   ├── quick-code-review.md          # Quick file review (7 invocations)
│   └── quick-code-review-staged.md   # Quick staged review (7 invocations)
├── agents/                      # Modular agent definitions (alphabetical)
│   ├── api-contracts-agent.md   # API compatibility
│   ├── architecture-agent.md    # Architecture patterns
│   ├── bug-detection-agent.md   # Logical errors & edge cases
│   ├── compliance-agent.md      # AI instructions compliance
│   ├── docs/                    # Documentation review agents
│   │   ├── accuracy-agent.md    # Code-doc sync, factual correctness
│   │   ├── clarity-agent.md     # Readability, jargon, audience
│   │   ├── completeness-agent.md # Missing sections, coverage
│   │   ├── consistency-agent.md # Terminology, formatting, style
│   │   ├── examples-agent.md    # Code example validity
│   │   └── structure-agent.md   # Organization, links, AI instructions
│   ├── error-handling-agent.md  # Error handling gaps
│   ├── performance-agent.md     # Performance issues
│   ├── security-agent.md        # Security vulnerabilities
│   ├── synthesis-agent.md       # Cross-agent insights
│   ├── technical-debt-agent.md  # Technical debt detection
│   └── test-coverage-agent.md   # Test coverage gaps
├── skills/                      # Targeted review skills
│   ├── architecture-principles-review/
│   ├── bug-review/
│   ├── compliance-review/
│   ├── docs-review/             # Documentation review skill
│   ├── performance-review/
│   ├── security-review/
│   └── technical-debt-review/
├── languages/                   # Language-specific configs
│   ├── nodejs.md                # Node.js/TypeScript checks
│   └── dotnet.md                # .NET/C# checks
├── shared/
│   ├── agent-common-instructions.md # Common agent instructions (MODE, gaps, pre-existing issue detection)
│   ├── agent-invocation-pattern.md  # Task invocation pattern for agents
│   ├── command-common-steps.md      # Common workflow steps for all commands
│   ├── content-gathering-docs.md    # Content gathering for documentation reviews
│   ├── content-gathering-files.md   # Content gathering for file reviews
│   ├── content-gathering-staged.md  # Content gathering for staged reviews
│   ├── context-discovery.md         # Context discovery instructions
│   ├── docs-agent-common-instructions.md # Common instructions for documentation agents
│   ├── docs-orchestration-sequence.md # Documentation review phase definitions
│   ├── input-validation-docs.md     # Input validation for documentation commands
│   ├── input-validation-files.md    # Input validation for file commands
│   ├── input-validation-staged.md   # Input validation for staged commands
│   ├── orchestration-sequence.md    # Phase definitions and model selection
│   ├── output-format.md             # Output schema, templates, and generation process
│   ├── settings-loader.md           # Settings loading and application
│   ├── severity-definitions.md      # Severity classification
│   ├── skill-orchestration.md       # Skill-informed orchestration (--skills)
│   ├── skill-resolver.md            # Skill resolution and parsing
│   ├── synthesis-invocation-pattern.md # Synthesis agent task pattern
│   ├── usage-tracking.md            # Usage tracking schema and protocol
│   ├── validation-rules.md          # Validation process
│   └── references/                  # Detailed reference content
│       ├── complete-output-example.md # Complete output format example
│       ├── scope-determination.md   # Scope options and edge cases
│       └── skill-troubleshooting.md # Common issues and solutions
└── README.md
```

### Agent Configuration

**Code Review Agents:**

| Agent | Model | Supported Modes | Color |
|-------|-------|-----------------|-------|
| api-contracts-agent | Sonnet | thorough | cyan |
| architecture-agent | Opus | thorough | yellow |
| bug-detection-agent | Opus | thorough, gaps, quick | red |
| compliance-agent | Sonnet | thorough, gaps | blue |
| error-handling-agent | Sonnet | thorough, quick | orange |
| performance-agent | Opus | thorough, gaps | green |
| security-agent | Opus | thorough, gaps, quick | purple |
| synthesis-agent | Sonnet | (cross-category) | white |
| technical-debt-agent | Opus | thorough, gaps | brown |
| test-coverage-agent | Sonnet | thorough, quick | white |

**Documentation Review Agents:**

| Agent | Model | Supported Modes | Color |
|-------|-------|-----------------|-------|
| accuracy-agent | Opus | thorough, gaps, quick | red |
| clarity-agent | Sonnet | thorough, quick | cyan |
| completeness-agent | Opus | thorough, gaps | green |
| consistency-agent | Sonnet | thorough, gaps | blue |
| examples-agent | Opus | thorough, quick | yellow |
| structure-agent | Sonnet | thorough, quick | purple |

> **Note:** Documentation reviews reuse `synthesis-agent` from the code review agents for cross-category analysis.

> **Note:** The `model` field in agent frontmatter is the default for standalone agent invocation (e.g., when Claude auto-selects an agent based on context). Commands may override this when invoking agents for specific modes—for example, using Sonnet for "gaps" mode to optimize cost while maintaining quality.

### MODE Parameter

Each agent accepts a MODE parameter:

| Mode | Description | Use Case |
|------|-------------|----------|
| **thorough** | Comprehensive review, check all issues | Default for full reviews |
| **gaps** | Focus on subtle issues that might be missed | Additional coverage for Opus agents |
| **quick** | Fast pass on critical issues only | Quick pre-commit checks |

### Review Configurations

**Code Reviews:**

| Command | Agents | Mode Invocations | Total Invocations |
|---------|--------|------------------|-------------------|
| `/deep-code-review` | 9 review + synthesis | thorough (9) + gaps (5) + synthesis (5) | 19 |
| `/deep-code-review-staged` | 9 review + synthesis | thorough (9) + gaps (5) + synthesis (5) | 19 |
| `/quick-code-review` | 4 (bugs, security, errors, tests) | quick (4) + synthesis (3) | 7 |
| `/quick-code-review-staged` | 4 (bugs, security, errors, tests) | quick (4) + synthesis (3) | 7 |

**Documentation Reviews:**

| Command | Agents | Mode Invocations | Total Invocations |
|---------|--------|------------------|-------------------|
| `/deep-docs-review` | All 6 docs | thorough (6) + gaps (3) + synthesis (4) | 13 |
| `/quick-docs-review` | 4 (accuracy, clarity, examples, structure) | quick (4) + synthesis (3) | 7 |

## Severity Levels

| Severity | Definition | Action |
|----------|------------|--------|
| **Critical** | Data loss, security breach, production outage risk | Must fix before merge |
| **Major** | Significant bug or violation affecting functionality | Should fix before merge |
| **Minor** | Small issue that should be addressed | Can merge, fix soon |
| **Suggestion** | Recommended improvement for future consideration | Optional |

## Language Support

### Node.js / TypeScript

Detected by presence of `package.json`. See `languages/nodejs.md` for:
- Bug patterns (promises, async/await, this binding)
- Security checks (prototype pollution, ReDoS, injection)
- Performance issues (event loop blocking, memory leaks)
- Architecture concerns (circular imports, hooks violations)
- Test file patterns

### .NET / C#

Detected by presence of `*.csproj` or `*.sln`. See `languages/dotnet.md` for:
- Bug patterns (null references, IDisposable, async deadlocks)
- Security checks (SQL injection, [Authorize], deserialization)
- Performance issues (boxing, LINQ in loops, N+1 queries)
- Architecture concerns (DI anti-patterns, controller bloat)
- Test file patterns

## Output Format

See `shared/output-format.md` for complete output templates.

### Summary Table

```markdown
## Code Review

**Reviewed:** 5 file(s) | **Branch:** feature/new-auth
**Review Depth:** Deep (19 invocations: 9 thorough + 5 gaps + 5 synthesis)

### Summary

| Category | Critical | Major | Minor | Suggestions |
|----------|----------|-------|-------|-------------|
| Bugs | 0 | 2 | 1 | 0 |
| Compliance | 0 | 1 | 0 | 0 |
| Security | 1 | 0 | 0 | 0 |
| ...
```

### Issue Format

```markdown
**1. SQL Injection Vulnerability** `Critical` `Security` [2 agents]
`src/data/UserRepository.cs:45-48`

The query uses string concatenation with user input.

```suggestion
var query = "SELECT * FROM Users WHERE Id = @id";
cmd.Parameters.AddWithValue("@id", userId);
```
```

## Workflow Integration

### Pre-commit Quick Review

```bash
git add src/feature.ts
/quick-code-review-staged
git commit -m "Add feature"
```

### Comprehensive Review Before PR

```bash
git add .
/deep-code-review-staged
```

### Targeted Security Audit

Use the security-review skill for focused security analysis.

### Legacy Code Audit

```bash
/deep-code-review src/legacy/critical-module.ts
```

## Customization

### Adding Language Support

Create a new file in `languages/` following the pattern of `nodejs.md` or `dotnet.md`.

### Modifying Agent Behavior

Edit the agent file in `agents/` to customize:
- Detection patterns
- Severity classification
- False positive rules

### Creating Custom Commands

Create a new `.md` file in `commands/` with the appropriate frontmatter.

## Requirements

- Git repository
- Claude Code CLI
- For staged reviews: Changes staged with `git add`
- For file reviews: Valid file paths

## Troubleshooting

### Common Issues

**Plugin not running:**
- Verify `.claude/code-review.local.md` has `enabled: true`
- Check file exists in project root

**No staged changes found:**
- Run `git add` before using staged commands
- Verify changes are staged with `git status`

**Wrong language detected:**
- Use `--language nodejs` or `--language dotnet` flag
- Or set `language` in settings file

**Too many findings:**
- Increase `min_severity` to "major" or "critical"
- Skip certain agents via `skip_agents` setting

For more troubleshooting, see `shared/references/skill-troubleshooting.md`.

## Version History

See [CHANGELOG.md](CHANGELOG.md) for full version history and release notes.

**Current Version:** 3.3.0

## Author

Jesse Naranjo

Based on the code-review plugin by Boris Cherny (boris@anthropic.com)
