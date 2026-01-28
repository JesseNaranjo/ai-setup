# Orchestration Sequence

This document defines the authoritative execution sequences for review pipelines.

## Deep Review Orchestration (19 agent invocations)

1. **Steps 1-3: Input, Context, Content**
   - Validate input, discover context, gather file content
   - OUTPUT: Files to review, diffs, AI instructions, test files

2. **Phase 1: Thorough Review** (9 agents in parallel)
   - Launch: api-contracts, architecture, bug-detection, compliance, error-handling, performance, security, technical-debt, test-coverage
   - Models: architecture, bug-detection, performance, security, technical-debt (Opus); api-contracts, compliance, error-handling, test-coverage (Sonnet)
   - MODE: `thorough` for all agents
   - WAIT: All 9 agents must complete before proceeding
   - OUTPUT: Phase 1 findings (grouped by category)

3. **Phase 2: Gaps Review** (5 Sonnet agents in parallel)
   - Launch: bug-detection, compliance, performance, security, technical-debt
   - MODE: `gaps`
   - Model: Sonnet (cost-optimized for constrained task)
   - INPUT: Phase 1 findings passed as `previous_findings`
   - WAIT: All 5 agents must complete before proceeding
   - OUTPUT: Phase 2 findings (subtle issues, edge cases)

4. **Synthesis** (5 agents in parallel)
   - Launch: 5 instances of synthesis-agent with category pairs
   - INPUT: ALL findings from Phase 1 AND Phase 2
   - Pairs and questions:
     - Architecture+Test Coverage: "Are architectural changes covered by tests?"
     - Bugs+Error Handling: "Do identified bugs have proper error handling in fix paths?"
     - Compliance+Bugs: "Do compliance violations introduce or mask bugs?"
     - Compliance+Technical Debt: "Do compliance violations indicate or worsen technical debt?"
     - Performance+Security: "Do any security fixes introduce performance issues?"
   - WAIT: All 5 must complete
   - OUTPUT: `cross_cutting_insights` list

5. **Validation, Aggregation, Output**
   - Validate all issues, filter invalid, deduplicate, generate report

## Quick Review Orchestration (7 agent invocations)

1. **Steps 1-3: Input, Context, Content** (same as deep)

2. **Review** (4 agents in parallel)
   - Launch: bug-detection, error-handling, security, test-coverage
   - MODE: `quick` for all agents
   - OUTPUT: Quick review findings

3. **Synthesis** (3 agents in parallel)
   - Pairs and questions:
     - Bugs+Error Handling: "Do identified bugs have proper error handling in fix paths?"
     - Bugs+Security: "Do security issues introduce or relate to bugs?"
     - Bugs+Test Coverage: "Are identified bugs covered by tests?"
   - OUTPUT: `cross_cutting_insights` list

4. **Validation, Aggregation, Output** (same as deep)

## Model Selection (Authoritative Source)

| Agent | Model (thorough) | Model (gaps) | Model (quick) |
|-------|------------------|--------------|---------------|
| api-contracts-agent | sonnet | N/A | N/A |
| architecture-agent | opus | N/A | N/A |
| bug-detection-agent | opus | sonnet | opus |
| compliance-agent | sonnet | sonnet | N/A |
| error-handling-agent | sonnet | N/A | sonnet |
| performance-agent | opus | sonnet | N/A |
| security-agent | opus | sonnet | opus |
| synthesis-agent | sonnet | N/A | sonnet |
| technical-debt-agent | opus | sonnet | N/A |
| test-coverage-agent | sonnet | N/A | sonnet |

## Language-Specific Focus

Load language configs ONLY for detected languages to minimize context usage:

- If `detected_languages.nodejs` has files: Load `${CLAUDE_PLUGIN_ROOT}/languages/nodejs.md`
- If `detected_languages.dotnet` has files: Load `${CLAUDE_PLUGIN_ROOT}/languages/dotnet.md`
- Skip loading configs for languages not present in the review

For mixed codebases (monorepos):
- Each file receives only its relevant language config
- Agents receive language-specific checks per file, not all configs
- Cross-language issues (e.g., API contract mismatches) are handled by architecture and API agents
