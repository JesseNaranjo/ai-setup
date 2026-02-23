# Code Review Plugin

Modular 10-agent code review with parameterized modes for Node.js, React, and .NET projects. Also includes documentation review commands with 7 specialized documentation agents.

## Overview

The Code Review Plugin provides automated, in-depth code review using 10 specialized agents that analyze code from different perspectives. Each agent supports multiple review modes (thorough, gaps, quick) for flexible review depth.

Additionally, the plugin provides documentation review commands that analyze project documentation for accuracy, clarity, completeness, consistency, examples quality, and structure.

### Key Features

- **Modular 10-Agent Architecture**: Each agent is a separate file in `agents/code/` or `agents/docs/` for easy customization
- **Parameterized Modes**: thorough, gaps, and quick modes for different review depths
- **Language-Aware**: Specialized checks for Node.js/TypeScript, React, and .NET/C# (configs in `languages/`)
- **Targeted Skills**: Security, performance, bug, compliance, and technical debt review skills
- **Validation Layer**: Every issue is independently validated to reduce false positives
- **Severity Classification**: Issues categorized as Critical, Major, Minor, or Suggestion
- **Consensus Scoring**: Issues flagged by multiple agents get higher confidence

## Commands

### `/code-review`

Code review with configurable depth. Describe what to review in the prompt: file paths, "staged changes", specific commits, or a description of the code area. Deep (default) uses all 9 review agents plus synthesis (up to 19 invocations) with thorough + gaps modes for maximum coverage. Quick uses 4 agents (up to 7 invocations) focusing on critical issues (bugs, security, errors, tests).

```bash
/code-review "<review prompt>" [--depth deep|quick] [--output-file <path>] [--language <nodejs|react|dotnet>] [--skills <skills>]
```

### `/docs-review`

Documentation review with configurable depth. Describe what to review in the prompt, or omit for auto-discovery of all project documentation (README.md, CLAUDE.md, docs/*, etc.). Deep (default) uses all 6 documentation agents (up to 13 invocations) with thorough + gaps modes. Quick uses 4 agents (up to 7 invocations) focusing on critical issues (accuracy, clarity, examples, structure).

```bash
/docs-review ["<review prompt>"] [--depth deep|quick] [--output-file <path>] [--skills <skills>]
```

> **Note:** Documentation commands support the `reviewing-documentation` skill which provides focused documentation quality checks across all 6 documentation agents (accuracy, clarity, completeness, consistency, examples, structure).

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
3. Review prompt appends to project instructions (does not replace)

### Review Prompt Strategies

The review prompt is your natural language interface to all review agents. Include both what to review and any special instructions directly in the prompt. This is useful for:
- Focusing reviews on specific concerns
- Providing one-off context for a particular review
- Experimenting with different review approaches

#### Example: Brainstorming-Style Deep Exploration

```bash
/code-review src/auth/*.ts - before flagging issues, brainstorm multiple attack vectors and failure modes for each function. What assumptions does this code make? What happens if those assumptions are violated? What edge cases might the original developer have missed? Explore creatively before concluding.
```

#### Example: Systematic Checklist Approach

```bash
/code-review review staged changes. For each file, systematically check: 1) Input validation gaps, 2) Error handling completeness, 3) Resource cleanup, 4) Concurrency issues, 5) Security boundaries. --depth quick
```

#### Example: Domain-Specific Focus

```bash
/code-review src/api/payments.ts - this is a payment processing module. Focus on: financial calculation precision, transaction atomicity, audit trail completeness, PCI compliance patterns. Flag anything that could lead to money loss or compliance violations.
```

#### Example: Threat Model Guidance

```bash
/code-review src/api/*.ts - assume an attacker has valid credentials but is trying to access other users' data. Focus on authorization checks, IDOR vulnerabilities, and data leakage in error messages.
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

Then add one-off instructions directly in the review prompt:

```bash
/code-review src/auth/*.ts - additionally, this PR introduces OAuth. Verify token handling follows OWASP guidelines.
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
| Plugin-local | `reviewing-security` | Skills within the code-review plugin |
| External plugin | `superpowers:brainstorming` | Skills from other plugins (plugin:skill) |

#### Examples

**Embed brainstorming methodology:**
```bash
/code-review src/auth/*.ts --skills superpowers:brainstorming
```
All agents will explore multiple interpretations and failure modes before flagging issues.

**Combine multiple skills:**
```bash
/code-review src/api/*.ts --skills reviewing-security,superpowers:systematic-debugging
```
Security agent receives targeted security checklists; all agents receive debugging methodology.

**Combine skills with review prompt:**
```bash
/code-review src/payments.ts - focus on financial calculation precision --skills superpowers:brainstorming
```
Combines skill methodology with specific instructions.

#### How It Works

1. Skill names are resolved to SKILL.md files
2. Skills are parsed into structured data (categories, patterns, rules, methodology)
3. The orchestrator generates tailored `skill_instructions` per agent:
   - **Review skills** (reviewing-security, etc.) → Targeted to their primary agent with focus areas and checklists
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
| `reviewing-architecture-principles` | architecture-agent | SOLID, DRY, YAGNI violations |
| `reviewing-bugs` | bug-detection-agent | Logical errors and edge cases |
| `reviewing-compliance` | compliance-agent | CLAUDE.md and coding standards |
| `reviewing-documentation` | all docs agents | Documentation quality, AI instructions |
| `reviewing-performance` | performance-agent | Performance issues and optimization |
| `reviewing-security` | security-agent | Security vulnerabilities and OWASP patterns |
| `reviewing-technical-debt` | technical-debt-agent | Deprecated code, outdated patterns, dead code |

## Skills

Targeted review skills for specific concerns:

| Skill | Trigger Phrases |
|-------|-----------------|
| `reviewing-architecture-principles` | "check SOLID principles", "review SOLID", "find SOLID violations", "check DRY", "find code duplication", "check YAGNI", "find over-engineering", "check SoC", "separation of concerns", "file organization" |
| `reviewing-bugs` | "find bugs", "check for bugs", "review for errors", "find logical errors", "check for null references", "find edge cases", "check for race conditions" |
| `reviewing-compliance` | "check CLAUDE.md compliance", "review against coding standards", "check AI agent instructions", "verify guidelines", "check coding conventions" |
| `reviewing-documentation` | "review documentation", "check docs", "audit README", "check CLAUDE.md", "verify AI instructions", "standardize docs", "review markdown" |
| `reviewing-performance` | "check performance", "review for performance issues", "find slow code", "optimize", "check for memory leaks", "find N+1 queries", "check complexity" |
| `reviewing-security` | "security review", "check for vulnerabilities", "audit security", "find security issues", "security scan", "check for injection", "find hardcoded secrets" |
| `reviewing-technical-debt` | "find technical debt", "check for deprecated code", "find outdated patterns", "identify dead code", "check for workarounds", "find TODO comments", "assess code health" |

## Architecture

### Directory Structure

```
code-review/
├── .claude-plugin/
│   └── plugin.json              # Plugin metadata
├── agents/                      # Modular agent definitions (10 code + 7 docs agents)
│   ├── code/                    # Code review agents (10 agents)
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
│   └── docs/                    # Documentation review agents (7 agents)
│       ├── accuracy-agent.md    # Code-doc sync, factual correctness
│       ├── clarity-agent.md     # Readability, jargon, audience
│       ├── completeness-agent.md # Missing sections, coverage
│       ├── consistency-agent.md # Terminology, formatting, style
│       ├── examples-agent.md    # Code example validity
│       ├── structure-agent.md   # Organization, links, AI instructions
│       └── synthesis-docs-agent.md  # Cross-agent insights (docs reviews)
├── skills/                      # Orchestration entry points and targeted review skills
│   ├── agent-review-instructions/   # Static agent config (MODE, FP rules, output schema)
│   ├── code-review/                # Orchestration: code review pipeline (deep: up to 19, quick: up to 7 invocations)
│   │   └── SKILL.md
│   ├── docs-review/                # Orchestration: docs review pipeline (deep: up to 13, quick: up to 7 invocations)
│   │   └── SKILL.md
│   ├── reviewing-architecture-principles/
│   ├── reviewing-bugs/
│   ├── reviewing-compliance/
│   ├── reviewing-documentation/  # Documentation review skill
│   ├── reviewing-performance/
│   ├── reviewing-security/
│   └── reviewing-technical-debt/
├── languages/                   # Language-specific configs
│   ├── dotnet.md                # .NET/C# checks
│   ├── nodejs.md                # Node.js/TypeScript checks
│   └── react.md                 # React checks (extends Node.js)
├── shared/
│   ├── review-orchestration-code.md # Code review: phases, invocation patterns, gaps mode behavior
│   ├── review-orchestration-docs.md # Docs review: phases, invocation patterns, gaps mode behavior
│   ├── review-validation-code.md    # Code validation: batch validation, aggregation, auto-validation patterns
│   ├── review-validation-docs.md    # Docs validation: batch validation, aggregation, auto-validation patterns
│   ├── skill-handling.md            # Skill resolution and orchestration (--skills)
│   └── references/                  # LSP integration details (progressive disclosure)
│       └── lsp-integration.md       # LSP integration details for Node.js and .NET
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
| synthesis-code-agent | Opus | (cross-category) | white |
| technical-debt-agent | Sonnet | thorough, gaps | brown |
| test-coverage-agent | Sonnet | thorough, quick | white |

**Documentation Review Agents:**

| Agent | Model | Supported Modes | Color |
|-------|-------|-----------------|-------|
| accuracy-agent | Opus | thorough, gaps, quick | red |
| clarity-agent | Sonnet | thorough, quick | cyan |
| completeness-agent | Sonnet | thorough, gaps | green |
| consistency-agent | Sonnet | thorough, gaps | blue |
| examples-agent | Sonnet | thorough, quick | yellow |
| structure-agent | Sonnet | thorough, quick | purple |
| synthesis-docs-agent | Opus | (cross-category) | white |

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

| Command | Depth | Agents | Mode Invocations | Total Invocations |
|---------|-------|--------|------------------|-------------------|
| `/code-review` | deep | 9 review + synthesis | thorough (9) + gaps (5) + synthesis (up to 5) | up to 19 |
| `/code-review` | quick | 4 (bugs, security, errors, tests) | quick (4) + synthesis (up to 3) | up to 7 |

> The review pipeline is the same regardless of scope type (file paths, staged changes, or descriptive scope).

**Documentation Reviews:**

| Command | Depth | Agents | Mode Invocations | Total Invocations |
|---------|-------|--------|------------------|-------------------|
| `/docs-review` | deep | All 6 docs | thorough (6) + gaps (3) + synthesis (up to 4) | up to 13 |
| `/docs-review` | quick | 4 (accuracy, clarity, examples, structure) | quick (4) + synthesis (up to 3) | up to 7 |

> Synthesis invocations = parallel instances of the synthesis agent, each analyzing a different cross-category pair. See Architecture section for pair definitions.

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

### React

Detected by `react` or `react-dom` in package.json dependencies. Extends Node.js checks. See `languages/react.md` for:
- Bug patterns (stale closures, dependency arrays, unmounted state updates)
- Security checks (XSS via dangerouslySetInnerHTML, javascript: URLs)
- Performance issues (missing memo/useCallback, inline objects, virtualization)
- Architecture concerns (prop drilling, component boundaries, hooks extraction)
- Test file patterns (testing-library patterns)

### .NET / C#

Detected by presence of `*.csproj` or `*.sln`. See `languages/dotnet.md` for:
- Bug patterns (null references, IDisposable, async deadlocks)
- Security checks (SQL injection, [Authorize], deserialization)
- Performance issues (boxing, LINQ in loops, N+1 queries)
- Architecture concerns (DI anti-patterns, controller bloat)
- Test file patterns

## Output Format

See `shared/review-validation-code.md` "Output Format" section for complete output templates.

### Summary Table

```markdown
## Code Review

**Reviewed:** 5 file(s) | **Branch:** feature/new-auth
**Review Depth:** Deep (up to 19 invocations: 9 thorough + 5 gaps + up to 5 synthesis)

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

**Fix**:
```diff
- var query = "SELECT * FROM Users WHERE Id = " + userId;
+ var query = "SELECT * FROM Users WHERE Id = @id";
+ cmd.Parameters.AddWithValue("@id", userId);
```
```

## Workflow Integration

### Pre-commit Quick Review

```bash
git add src/feature.ts
/code-review review staged changes --depth quick
git commit -m "Add feature"
```

### Comprehensive Review Before PR

```bash
git add .
/code-review review staged changes
```

### Post-Commit Review

```bash
/code-review review the last commit --depth quick
```

### Review a Commit Range

```bash
/code-review review changes since main
```

### Targeted Security Audit

Use the reviewing-security skill for focused security analysis.

### Legacy Code Audit

```bash
/code-review src/legacy/critical-module.ts
```

## Customization

### Adding Language Support

Create a new file in `languages/` following the pattern of `nodejs.md` or `dotnet.md`.

### Modifying Agent Behavior

Edit the agent file in `agents/code/` or `agents/docs/` to customize:
- Detection patterns
- Severity classification
- False positive rules

### Creating Custom Skills

Create a new directory in `skills/` with a `SKILL.md` file containing the appropriate frontmatter.

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
- Run `git add` before reviewing staged changes
- Verify changes are staged with `git status`

**Wrong language detected:**
- Use `--language nodejs`, `--language react`, or `--language dotnet` flag
- Or set `language` in settings file
- Run review from the specific project subdirectory
- Check if both package.json and .csproj exist in the repo (mixed project)
- Check if files are in a subdirectory with different project type

**Too many findings:**
- Increase `min_severity` to "major" or "critical"
- Skip certain agents via `skip_agents` setting

**Agent returns no issues but problems exist:**
- Verify file content is included in the prompt
- Check that diffs show the problematic lines
- Ensure project_type is correctly detected
- Try expanding scope to include related files
- Re-run with explicit file paths instead of staged changes
- Check if file is in .gitignore (won't appear in staged)
- Verify the issue is in changed lines (not pre-existing)

**Too many false positives:**
- Are issues in test files? (often intentional)
- Are issues in commented code?
- Are "vulnerabilities" in internal-only code?
- Increase min_severity to "Major" in settings
- Exclude test files if not relevant
- Add context about intentional exceptions in project settings

**Agent times out:**
- Reduce scope to fewer files
- Split large files into focused reviews
- Use quick mode for initial scan, then deep review specific areas

**Skill not triggering:**
- Verify plugin is listed in `/help`
- Use exact trigger phrases (see Skills section above)
- Restart Claude Code to reload plugins

**Cross-cutting issues missed:**
- Use `/code-review` which includes synthesis phase
- Explicitly ask: "Check if security fixes affect performance"
- Review related files together, not in isolation

## Version History

See [CHANGELOG.md](CHANGELOG.md) for full version history and release notes.

**Current Version:** 4.0.1

## Author

Jesse Naranjo

Based on the code-review plugin by Boris Cherny (boris@anthropic.com)
