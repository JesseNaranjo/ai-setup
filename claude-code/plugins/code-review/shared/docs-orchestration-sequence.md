# Documentation Review Orchestration Sequence

This document defines the authoritative execution sequences for documentation review pipelines.

## Deep Docs Review Orchestration (13 agent invocations)

1. **Steps 1-3: Input, Context, Content**
   - Discover and validate documentation files
   - Gather doc content and related code references
   - OUTPUT: Documentation files, related code snippets, AI instruction files

2. **Phase 1: Thorough Review** (6 agents in parallel)
   - Launch: accuracy, clarity, completeness, consistency, examples, structure
   - Models: accuracy, completeness, examples (Opus); clarity, consistency, structure (Sonnet)
   - MODE: `thorough` for all agents
   - WAIT: All 6 agents must complete before proceeding
   - OUTPUT: Phase 1 findings (grouped by category)

3. **Phase 2: Gaps Review** (3 Sonnet agents in parallel)
   - Launch: accuracy, completeness, consistency
   - MODE: `gaps`
   - Model: Sonnet (cost-optimized for constrained task)
   - INPUT: Phase 1 findings passed as `previous_findings`
   - WAIT: All 3 agents must complete before proceeding
   - OUTPUT: Phase 2 findings (subtle issues, edge cases)

4. **Synthesis** (4 agents in parallel)
   - Launch: 4 instances of synthesis-agent with category pairs
   - INPUT: ALL findings from Phase 1 AND Phase 2
   - Pairs and questions:
     - Accuracy+Examples: "Do code examples match the documented behavior they claim to demonstrate?"
     - Clarity+Structure: "Does poor structure contribute to clarity issues, or vice versa?"
     - Completeness+Consistency: "Are missing sections causing terminology inconsistencies elsewhere?"
     - Consistency+Structure: "Do formatting inconsistencies reflect structural organization problems?"
   - WAIT: All 4 must complete
   - OUTPUT: `cross_cutting_insights` list

5. **Validation, Aggregation, Output**
   - Validate all issues, filter invalid, deduplicate, generate report

## Quick Docs Review Orchestration (7 agent invocations)

1. **Steps 1-3: Input, Context, Content** (same as deep)

2. **Review** (4 agents in parallel)
   - Launch: accuracy, clarity, examples, structure
   - MODE: `quick` for all agents
   - OUTPUT: Quick review findings

3. **Synthesis** (3 agents in parallel)
   - Pairs and questions:
     - Accuracy+Examples: "Do code examples match the documented behavior they claim to demonstrate?"
     - Clarity+Structure: "Does poor structure contribute to clarity issues, or vice versa?"
     - Examples+Structure: "Are example placements and references structurally sound?"
   - OUTPUT: `cross_cutting_insights` list

4. **Validation, Aggregation, Output** (same as deep)

## Model Selection (Authoritative Source)

| Agent | Model (thorough) | Model (gaps) | Model (quick) |
|-------|------------------|--------------|---------------|
| accuracy-agent | opus | sonnet | opus |
| clarity-agent | sonnet | N/A | sonnet |
| completeness-agent | opus | sonnet | N/A |
| consistency-agent | sonnet | sonnet | N/A |
| examples-agent | opus | N/A | opus |
| structure-agent | sonnet | N/A | sonnet |
| synthesis-agent | sonnet | N/A | sonnet |

## Agent Focus Areas

### accuracy-agent
- Code-documentation synchronization
- Factual correctness of technical claims
- Version and API accuracy
- Command/CLI documentation validity

### clarity-agent
- Readability and comprehension
- Jargon and technical term usage
- Audience appropriateness
- Explanation quality

### completeness-agent
- Missing required sections
- Undocumented features/APIs
- Installation/setup gaps
- Troubleshooting coverage

### consistency-agent
- Terminology uniformity
- Formatting standards
- Voice and tone consistency
- Naming convention adherence

### examples-agent
- Code example correctness
- Example completeness (imports, context)
- Example relevance to documentation
- Output/result accuracy

### structure-agent
- Document organization
- Navigation and discoverability
- Link integrity (internal and external)
- Hierarchy and heading structure
- **AI instruction file standardization** (see below)

## AI Instruction File Standardization

The structure-agent includes standardization checks for AI agent instruction files:

### Required Files and Locations
- `/.ai/AI-AGENT-INSTRUCTIONS.md` - Comprehensive coding standards
- `/CLAUDE.md` - Claude Code quick reference (references .ai/AI-AGENT-INSTRUCTIONS.md)
- `/.github/copilot-instructions.md` - GitHub Copilot quick reference (references .ai/AI-AGENT-INSTRUCTIONS.md)

### Standardization Rules
1. AI-AGENT-INSTRUCTIONS.md must be in `/.ai/` directory (not root)
2. CLAUDE.md must have header referencing `.ai/AI-AGENT-INSTRUCTIONS.md`
3. copilot-instructions.md must exist with header referencing `.ai/AI-AGENT-INSTRUCTIONS.md`
4. Cross-references between all three files must be valid

See `${CLAUDE_PLUGIN_ROOT}/skills/docs-review/references/ai-instruction-templates.md` for required header templates.
