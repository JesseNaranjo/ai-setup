# Orchestration Sequence

This document defines the authoritative execution sequences for review pipelines.

## Deep Review Orchestration (18 agent invocations)

1. **Steps 1-3: Input, Context, Content**
   - Validate input, discover context, gather file content
   - OUTPUT: Files to review, diffs, AI instructions, test files

2. **Phase 1: Thorough Review** (9 agents in parallel)
   - Launch: bug, security, performance, technical-debt (Opus) + compliance, architecture, api, error-handling, test-coverage (Sonnet)
   - MODE: `thorough` for all agents
   - WAIT: All 8 agents must complete before proceeding
   - OUTPUT: Phase 1 findings (grouped by category)

3. **Phase 2: Gaps Review** (5 Sonnet agents in parallel)
   - Launch: compliance, bug, security, performance, technical-debt
   - MODE: `gaps`
   - Model: Sonnet (cost-optimized for constrained task)
   - INPUT: Phase 1 findings passed as `previous_findings`
   - WAIT: All 4 agents must complete before proceeding
   - OUTPUT: Phase 2 findings (subtle issues, edge cases)

4. **Synthesis** (4 agents in parallel)
   - Launch: 4 instances of synthesis-agent with category pairs
   - INPUT: ALL findings from Phase 1 AND Phase 2
   - Pairs: Security+Performance, Architecture+Test Coverage, Bugs+Error Handling, Technical Debt+Compliance
   - WAIT: All 4 must complete
   - OUTPUT: `cross_cutting_insights` list

5. **Validation, Aggregation, Output**
   - Validate all issues, filter invalid, deduplicate, generate report

## Quick Review Orchestration (7 agent invocations)

1. **Steps 1-3: Input, Context, Content** (same as deep)

2. **Review** (4 agents in parallel)
   - Launch: bug, security, error-handling, test-coverage
   - MODE: `quick` for all agents
   - OUTPUT: Quick review findings

3. **Synthesis** (3 agents in parallel)
   - Pairs: Bugs+Error Handling, Security+Bugs, Bugs+Test Coverage
   - OUTPUT: `cross_cutting_insights` list

4. **Validation, Aggregation, Output** (same as deep)

## Model Selection (Authoritative Source)

| Agent | Model (thorough) | Model (gaps) | Model (quick) |
|-------|------------------|--------------|---------------|
| compliance-agent | sonnet | sonnet | N/A |
| bug-detection-agent | opus | sonnet | opus |
| security-agent | opus | sonnet | opus |
| performance-agent | opus | sonnet | N/A |
| architecture-agent | sonnet | N/A | N/A |
| api-contracts-agent | sonnet | N/A | N/A |
| error-handling-agent | sonnet | N/A | sonnet |
| test-coverage-agent | sonnet | N/A | sonnet |
| technical-debt-agent | opus | sonnet | N/A |
| synthesis-agent | sonnet | N/A | sonnet |
