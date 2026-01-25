# Code Review Plugin

Modular 9-agent code review with parameterized modes for Node.js and .NET projects.

## Overview

The Code Review Plugin provides automated, in-depth code review using 9 specialized agents that analyze code from different perspectives. Each agent supports multiple review modes (thorough, gaps, quick) for flexible review depth.

### Key Features

- **Modular 9-Agent Architecture**: Each agent is a separate file in `agents/` for easy customization
- **Parameterized Modes**: thorough, gaps, and quick modes for different review depths
- **Language-Aware**: Specialized checks for Node.js/TypeScript and .NET/C# (configs in `languages/`)
- **Targeted Skills**: Security, performance, bug, and compliance review skills
- **Validation Layer**: Every issue is independently validated to reduce false positives
- **Severity Classification**: Issues categorized as Critical, Major, Minor, or Suggestion
- **Consensus Scoring**: Issues flagged by multiple agents get higher confidence

## Commands

### `/deep-review`

Comprehensive code review using all 9 agents (16 invocations) with thorough + gaps modes for maximum coverage.

```bash
/deep-review <file1> [file2...] [--output-file <path>] [--language <nodejs|dotnet>] [--prompt "<instructions>"] [--skills <skills>]
```

### `/deep-review-staged`

Comprehensive code review of staged git changes using all 9 agents (16 invocations) with thorough + gaps modes.

```bash
/deep-review-staged [--output-file <path>] [--language <nodejs|dotnet>] [--prompt "<instructions>"] [--skills <skills>]
```

### `/quick-review`

Fast 4-agent review of specific files focusing on critical issues (bugs, security, errors, tests).

```bash
/quick-review <file1> [file2...] [--output-file <path>] [--language <nodejs|dotnet>] [--prompt "<instructions>"] [--skills <skills>]
```

### `/quick-review-staged`

Fast 4-agent review of staged git changes focusing on critical issues (bugs, security, errors, tests).

```bash
/quick-review-staged [--output-file <path>] [--language <nodejs|dotnet>] [--prompt "<instructions>"] [--skills <skills>]
```

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

# Agents to skip (options: compliance, bugs, security, performance,
#                         architecture, api-contracts, error-handling, test-coverage)
skip_agents: []

# Minimum severity to report (options: critical, major, minor, suggestion)
min_severity: "suggestion"

# Language override (options: nodejs, dotnet, or empty for auto-detect)
language: ""

# Custom rules (checked by compliance agent)
custom_rules: []
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
| `custom_rules` | `[]` | Additional rules for compliance checks |

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

**Add custom rules:**
```yaml
---
custom_rules:
  - pattern: "console\\.log"
    message: "Remove console.log before committing"
    severity: "minor"
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
/deep-review src/auth/*.ts --prompt "Before flagging issues, brainstorm multiple attack vectors and failure modes for each function. Consider: What assumptions does this code make? What happens if those assumptions are violated? What edge cases might the original developer have missed? Explore creatively before concluding."
```

#### Example: Systematic Checklist Approach

```bash
/quick-review-staged --prompt "For each file, systematically check: 1) Input validation gaps, 2) Error handling completeness, 3) Resource cleanup, 4) Concurrency issues, 5) Security boundaries. Don't skip any category even if it seems unlikely."
```

#### Example: Domain-Specific Focus

```bash
/deep-review src/api/payments.ts --prompt "This is a payment processing module. Focus on: financial calculation precision, transaction atomicity, audit trail completeness, PCI compliance patterns. Flag anything that could lead to money loss or compliance violations."
```

#### Example: Threat Model Guidance

```bash
/deep-review src/api/*.ts --prompt "Assume an attacker has valid credentials but is trying to access other users' data. Focus on authorization checks, IDOR vulnerabilities, and data leakage in error messages."
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
/deep-review src/auth/*.ts --prompt "Additionally, this PR introduces OAuth. Verify token handling follows OWASP guidelines."
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
/deep-review src/auth/*.ts --skills superpowers:brainstorming
```
All agents will explore multiple interpretations and failure modes before flagging issues.

**Combine multiple skills:**
```bash
/deep-review src/api/*.ts --skills security-review,superpowers:systematic-debugging
```
Security agent receives targeted security checklists; all agents receive debugging methodology.

**Use with --prompt:**
```bash
/deep-review src/payments.ts --skills superpowers:brainstorming --prompt "Focus on financial calculation precision"
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
| `security-review` | security-agent | Security vulnerabilities and OWASP patterns |
| `performance-review` | performance-agent | Performance issues and optimization |
| `bug-review` | bug-detection-agent | Logical errors and edge cases |
| `compliance-review` | compliance-agent | CLAUDE.md and coding standards |

## Skills

Targeted review skills for specific concerns:

| Skill | Trigger Phrases |
|-------|-----------------|
| `security-review` | "security review", "check for vulnerabilities", "audit security" |
| `performance-review` | "check performance", "find slow code", "optimize" |
| `bug-review` | "find bugs", "check for errors", "find edge cases" |
| `compliance-review` | "check CLAUDE.md compliance", "review against standards" |

## Architecture

### Directory Structure

```
code-review/
├── .claude-plugin/
│   └── plugin.json              # Plugin metadata (v3.1.4)
├── commands/
│   ├── deep-review.md           # Deep file review (16 invocations)
│   ├── deep-review-staged.md    # Deep staged review (16 invocations)
│   ├── quick-review.md          # Quick file review (7 invocations)
│   └── quick-review-staged.md   # Quick staged review (7 invocations)
├── agents/                      # Modular agent definitions
│   ├── compliance-agent.md      # AI instructions compliance
│   ├── bug-detection-agent.md   # Logical errors & edge cases
│   ├── security-agent.md        # Security vulnerabilities
│   ├── performance-agent.md     # Performance issues
│   ├── architecture-agent.md    # Architecture patterns
│   ├── api-contracts-agent.md   # API compatibility
│   ├── error-handling-agent.md  # Error handling gaps
│   ├── test-coverage-agent.md   # Test coverage gaps
│   └── synthesis-agent.md       # Cross-agent insights
├── skills/                      # Targeted review skills
│   ├── security-review/
│   ├── performance-review/
│   ├── bug-review/
│   └── compliance-review/
├── languages/                   # Language-specific configs
│   ├── nodejs.md                # Node.js/TypeScript checks
│   └── dotnet.md                # .NET/C# checks
├── shared/
│   ├── content-gathering-files.md   # Content gathering for file reviews
│   ├── content-gathering-staged.md  # Content gathering for staged reviews
│   ├── context-discovery.md         # Context discovery instructions
│   ├── false-positives.md           # False positive rules
│   ├── gaps-mode-rules.md           # Rules for gaps mode operation
│   ├── input-validation-files.md    # Input validation for file commands
│   ├── input-validation-staged.md   # Input validation for staged commands
│   ├── output-format.md             # Output templates
│   ├── output-generation.md         # Output generation and file writing
│   ├── output-schema-base.md        # Base YAML schema for all agents
│   ├── review-workflow.md           # Orchestration logic
│   ├── settings-loader.md           # Settings loading and application
│   ├── severity-definitions.md      # Severity classification
│   ├── skill-common-workflow.md     # Lean skill workflow (references details)
│   ├── validation-rules.md          # Validation process
│   └── references/                  # Detailed reference content
│       ├── scope-determination.md   # Scope options and edge cases
│       ├── validation-details.md    # Batch validation procedures
│       └── skill-troubleshooting.md # Common issues and solutions
└── README.md
```

### Agent Configuration

| Agent | Model | Supported Modes | Color |
|-------|-------|-----------------|-------|
| compliance-agent | Sonnet | thorough, gaps | blue |
| bug-detection-agent | Opus | thorough, gaps, quick | red |
| security-agent | Opus | thorough, gaps, quick | magenta |
| performance-agent | Opus | thorough, gaps | yellow |
| architecture-agent | Sonnet | thorough | cyan |
| api-contracts-agent | Sonnet | thorough | green |
| error-handling-agent | Sonnet | thorough, quick | orange |
| test-coverage-agent | Sonnet | thorough, quick | purple |
| synthesis-agent | Sonnet | (cross-category) | white |

> **Note:** The `model` field in agent frontmatter is the default for standalone agent invocation (e.g., when Claude auto-selects an agent based on context). Commands may override this when invoking agents for specific modes—for example, using Sonnet for "gaps" mode to optimize cost while maintaining quality.

### MODE Parameter

Each agent accepts a MODE parameter:

| Mode | Description | Use Case |
|------|-------------|----------|
| **thorough** | Comprehensive review, check all issues | Default for full reviews |
| **gaps** | Focus on subtle issues that might be missed | Additional coverage for Opus agents |
| **quick** | Fast pass on critical issues only | Quick pre-commit checks |

### Review Configurations

| Command | Agents | Mode Invocations | Total Invocations |
|---------|--------|------------------|-------------------|
| `/deep-review` | All 9 | thorough (8) + gaps (4) + synthesis (4) | 16 |
| `/deep-review-staged` | All 9 | thorough (8) + gaps (4) + synthesis (4) | 16 |
| `/quick-review` | 4 (bugs, security, errors, tests) | quick (4) + synthesis (3) | 7 |
| `/quick-review-staged` | 4 (bugs, security, errors, tests) | quick (4) + synthesis (3) | 7 |

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
**Review Depth:** Deep (16 invocations: 8 thorough + 4 gaps + 4 synthesis)

### Summary

| Category | Critical | Major | Minor | Suggestions |
|----------|----------|-------|-------|-------------|
| Compliance | 0 | 1 | 0 | 0 |
| Bugs | 0 | 2 | 1 | 0 |
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
/quick-review-staged
git commit -m "Add feature"
```

### Comprehensive Review Before PR

```bash
git add .
/deep-review-staged
```

### Targeted Security Audit

Use the security-review skill for focused security analysis.

### Legacy Code Audit

```bash
/deep-review src/legacy/critical-module.ts
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

**Current Version:** 3.1.4

## Author

Jesse Naranjo

Based on the code-review plugin by Boris Cherny (boris@anthropic.com)
