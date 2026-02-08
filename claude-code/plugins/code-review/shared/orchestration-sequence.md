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
   - **CRITICAL: WAIT** - DO NOT proceed to Phase 2 until ALL 9 agents complete
   - OUTPUT: Phase 1 findings (grouped by category)

3. **Phase 2: Gaps Review** (5 Sonnet agents in parallel)
   - Launch: bug-detection, compliance, performance, security, technical-debt
   - MODE: `gaps`
   - Model: Sonnet (cost-optimized for constrained task)
   - INPUT: Phase 1 findings passed as `previous_findings`
   - **CRITICAL: WAIT** - DO NOT proceed to Synthesis until ALL 5 agents complete
   - OUTPUT: Phase 2 findings (subtle issues, edge cases)

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

4. **Synthesis** (5 agents in parallel)
   - **CRITICAL: DO NOT START until Phase 1 AND Phase 2 are FULLY COMPLETE**
   - Launch: 5 instances of synthesis-code-agent with category pairs
   - INPUT: ALL findings from Phase 1 AND Phase 2
   - Pairs and questions:
     - Architecture+Test Coverage: "Are architectural changes covered by tests?"
     - Bugs+Compliance: "Do compliance violations introduce or mask bugs?"
     - Bugs+Error Handling: "Do identified bugs have proper error handling in fix paths?"
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

## Code Review Model Selection (Authoritative Source)

| Agent | Model (thorough) | Model (gaps) | Model (quick) |
|-------|------------------|--------------|---------------|
| api-contracts-agent | sonnet | N/A | N/A |
| architecture-agent | opus | N/A | N/A |
| bug-detection-agent | opus | sonnet | opus |
| compliance-agent | sonnet | sonnet | N/A |
| error-handling-agent | sonnet | N/A | sonnet |
| performance-agent | opus | sonnet | N/A |
| security-agent | opus | sonnet | opus |
| synthesis-code-agent | sonnet | N/A | sonnet |
| technical-debt-agent | opus | sonnet | N/A |
| test-coverage-agent | sonnet | N/A | sonnet |

## Content Strategy by Phase

Different phases have different content requirements. This strategy reduces token usage while maintaining review quality.

**Phase 1 (Thorough):**
- Full file content provided
- Full diff content provided
- AI instructions per distribution rules (see invocation-patterns.md)
- Test files per distribution rules (see invocation-patterns.md)
- Agents perform comprehensive analysis

**Phase 2 (Gaps):**
- Diff content always provided
- Full file content: Not provided by default - agents use Read tool if deeper analysis needed
- Previous findings provided (defines skip zones)
- Focus: Subtle issues, edge cases not caught in Phase 1
- Model: Sonnet (cost-optimized for constrained task with prior context)

**Synthesis:**
- Findings from Phase 1 + Phase 2 provided
- File paths provided (agents can Read if cross-reference needed)
- Focus: Cross-cutting concerns only
- No file content passed by default

**Rationale:** Agents have access to Read, Grep, and Glob tools. Providing file paths instead of full content for later phases allows agents to fetch content on-demand, reducing baseline token usage while preserving capability.

## Language-Specific Focus

Load language configs ONLY for detected languages/frameworks to minimize context usage:

- If `detected_languages.dotnet` has files: Load `${CLAUDE_PLUGIN_ROOT}/languages/dotnet.md`
- If `detected_languages.nodejs` has files: Load `${CLAUDE_PLUGIN_ROOT}/languages/nodejs.md`
- If `detected_frameworks.react` has files: Also load `${CLAUDE_PLUGIN_ROOT}/languages/react.md`
- Skip loading configs for languages/frameworks not present in the review

For mixed codebases (monorepos):
- Each file receives only its relevant language config
- React files receive both nodejs.md AND react.md checks
- Agents receive language-specific checks per file, not all configs
- Cross-language issues (e.g., API contract mismatches) are handled by architecture and API agents

## Documentation Review Orchestration

### Deep Docs Review Orchestration (13 agent invocations)

1. **Steps 1-3: Input, Context, Content**
   - Discover and validate documentation files
   - Gather doc content and related code references
   - OUTPUT: Documentation files, related code snippets, AI instruction files

2. **Phase 1: Thorough Review** (6 agents in parallel)
   - Launch: accuracy, clarity, completeness, consistency, examples, structure
   - Models: accuracy, completeness, examples (Opus); clarity, consistency, structure (Sonnet)
   - MODE: `thorough` for all agents
   - **CRITICAL: WAIT** - DO NOT proceed to Phase 2 until ALL 6 agents complete
   - OUTPUT: Phase 1 findings (grouped by category)

3. **Phase 2: Gaps Review** (3 Sonnet agents in parallel)
   - Launch: accuracy, completeness, consistency
   - MODE: `gaps`
   - Model: Sonnet (cost-optimized for constrained task)
   - INPUT: Phase 1 findings passed as `previous_findings`
   - **CRITICAL: WAIT** - DO NOT proceed to Synthesis until ALL 3 agents complete
   - OUTPUT: Phase 2 findings (subtle issues, edge cases)

4. **Synthesis** (4 agents in parallel)
   - **CRITICAL: DO NOT START until Phase 1 AND Phase 2 are FULLY COMPLETE**
   - Launch: 4 instances of synthesis-docs-agent with category pairs
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

### Quick Docs Review Orchestration (7 agent invocations)

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

### Documentation Review Model Selection (Authoritative Source)

| Agent | Model (thorough) | Model (gaps) | Model (quick) |
|-------|------------------|--------------|---------------|
| accuracy-agent | opus | sonnet | opus |
| clarity-agent | sonnet | N/A | sonnet |
| completeness-agent | opus | sonnet | N/A |
| consistency-agent | sonnet | sonnet | N/A |
| examples-agent | opus | N/A | opus |
| structure-agent | sonnet | N/A | sonnet |
| synthesis-docs-agent | sonnet | N/A | sonnet |
