# Changelog

All notable changes to the Code Review Plugin are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [3.2.2] - 2026-01-28

### Added
- **Architecture Principles Review Skill**: New skill for detecting SOLID, DRY, and YAGNI violations with severity mappings
  - `skills/architecture-principles-review/SKILL.md` - Core skill workflow (122 lines)
  - `skills/architecture-principles-review/references/solid-dry-yagni-patterns.md` - Detection patterns (291 lines)
  - `skills/architecture-principles-review/examples/example-output.md` - Example findings (102 lines)
- **Synthesis Invocation Pattern**: New `shared/synthesis-invocation-pattern.md` documenting 5-way parallel synthesis invocation
- **Complete Output Example**: New `shared/references/complete-output-example.md` with comprehensive output format reference
- **Pre-Existing Issue Detection**: Added section to `agent-common-instructions.md` with rules for flagging issues from staged/diff reviews
- **Automatic Cross-File Analysis**: Added section to `agent-common-instructions.md` for cross-cutting concern detection

### Changed
- **Agent description format**: All 10 agents now have 3-example format with Context, User, Assistant, and Commentary sections
- **Synthesis agent refactored**: Reduced from 224 lines to lean template using shared synthesis-invocation-pattern.md
- **Output format consolidated**: Merged output generation process into `output-format.md` (step-by-step workflow)
- **Skill descriptions standardized**: All 7 skills use consistent third-person trigger phrase format
- **Command documentation**: All 4 commands updated to reference new shared patterns
- **Orchestration sequence**: Added language-specific focus guidance for selective config loading in monorepos
- **Validation rules**: Enhanced cross-references and consensus detection

### Removed
- `shared/gaps-mode-rules.md` - content consolidated into `agent-common-instructions.md`
- `shared/output-schema-base.md` - content consolidated into `output-format.md`
- `shared/skill-common-workflow.md` - content distributed to individual skills
- `shared/output-generation.md` - content consolidated into `output-format.md`
- `shared/review-workflow.md` - content distributed to specialized documents

### Fixed
- Alphabetical ordering violation in synthesis pairs (Compliance+Bugs → Bugs+Compliance)
- Agent invocation pattern references updated to use `${CLAUDE_PLUGIN_ROOT}` consistently
- Skill resolver clarified Skill() tool as primary loading method
- README updated with accurate 10-agent architecture and 19-invocation deep review pipeline

## [3.2.1] - 2026-01-26

### Fixed
- CHANGELOG Version Locations skill count (4 → 5 files)
- Command description inconsistency in quick-code-review-staged (added "code")
- CLAUDE.md version count documentation (accurate file breakdown: ~21 files)

### Changed
- Moved tiered context explanation from staged commands to `content-gathering-staged.md`
- Consolidated workflow step numbering with clear scheme in `command-common-steps.md` (Steps 1, 3, 5, 8-11)
- Added cross-reference note to `review-workflow.md` clarifying relationship with command steps
- Added "Related Files" sections to `agent-invocation-pattern.md` and `output-format.md`
- Standardized all path references to use `${CLAUDE_PLUGIN_ROOT}/shared/...` format consistently

## [3.2.0] - 2026-01-26

### Added
- **Technical Debt Agent**: New 10th agent for detecting deprecated dependencies, outdated patterns, dead code, and TODO/FIXME accumulation
- **Technical Debt Review Skill**: New `technical-debt-review` skill with comprehensive patterns and examples
- **Language-specific technical debt patterns**: Added Node.js and .NET specific technical debt checks

### Changed
- **10-agent architecture**: Upgraded from 9-agent to 10-agent architecture
- **Orchestration sequences**: Updated deep review to include technical debt agent (Phase 1: 9 agents, Phase 2: 5 agents)
- **DRY optimization**: Consolidated duplicated content to authoritative sources (~2,250 tokens saved per deep review):
  - Replaced inline false positive rules with reference to `shared/false-positives.md`
  - Replaced inline phase descriptions with reference to `shared/orchestration-sequence.md`
  - Deleted redundant `shared/references/validation-details.md` (content in `validation-rules.md`)
  - Updated `skill-common-workflow.md` reference

### Fixed
- Added missing test code exclusion rule to `shared/false-positives.md`

## [3.1.4] - 2026-01-25

### Changed
- Refactored shared documentation to reduce context window usage (~35% reduction)
- Created `shared/orchestration-sequence.md` for phase definitions and model selection
- Created `shared/agent-invocation-pattern.md` for Task invocation template
- Expanded `shared/agent-common-instructions.md` with MODE, false positives, gaps mode, and output schema
- Updated all 9 agent files to reference common instructions instead of embedding duplicates
- Updated all 4 command files to reference new shared files
- Refactored `shared/review-workflow.md` to use references (783 → 506 lines)

### Fixed
- Eliminated duplicated content across agent files

## [3.1.3] - 2025-01-25

### Fixed
- **Removed `engines` field** from plugin.json as it is not supported in Claude Code plugins and triggers a manifest validation error

## [3.1.2] - 2025-01-25

### Added
- **Skill-informed orchestration**: Comprehensive documentation for how `--skills` affects agent prompt generation, validation, and synthesis phases
- **New shared documentation files**:
  - `shared/skill-loading.md`: Centralized skill loading process requiring Skill() tool as primary method
  - `shared/skill-instructions-usage.md`: How agents apply `skill_instructions` (focus_areas, checklists, methodology)
  - `shared/content-gathering-common.md`: Common steps for reading related test files
- **Security review skill patterns**: 5 new high-confidence auto-validated patterns:
  - `hardcoded_token`, `hardcoded_secret`, `hardcoded_credentials`
  - `sql_injection_template`, `new_function_untrusted`
- **Consensus detection algorithm**: Detailed 5-step algorithm with overlap detection formulas and merge examples
- **`engines` field**: Added to plugin.json for Claude Code version compatibility

### Changed
- **Skill resolver**: Now mandates Skill() tool as primary loading method with direct file read as fallback only
- **All 9 agents**: Added `skill_instructions` support documentation for applying skill-derived focus areas and methodology
- **Gaps mode rules**: Enhanced skip zone calculations with explicit formulas and overlap detection examples
- **Commands**: Refactored to reference shared skill-loading.md and review-workflow.md, reducing duplication (~280 lines removed)
- **Validation rules**: Added cross-reference to false-positives.md for complete false positive patterns

### Fixed
- CHANGELOG placeholder dates (2025-01-XX) replaced with actual release dates

## [3.1.1] - 2025-01-23

### Fixed
- README directory structure comment showing outdated version (v3.0.3 → v3.1.1)
- README Agent Configuration table: corrected supported modes for agents not used in quick review
- README Agent Configuration table: fixed synthesis-agent color from cyan to white

## [3.1.0] - 2025-01-23

### Added
- **`--skills` option**: Embed skill methodologies in agent prompts (e.g., `--skills security-review,superpowers:brainstorming`)
- **`--prompt` option**: Pass additional instructions to review agents
- **Skill resolver**: New `shared/skill-resolver.md` for resolving skill names to SKILL.md files
- **Usage tracking protocol**: New `shared/usage-tracking.md` for agent invocation timing and anomaly detection

### Changed
- Synthesis agent color changed to white for visual distinction
- Enhanced model selection tables with quick mode mappings
- Added synthesis invocation examples to review workflow documentation
- Added agent color identification to review documentation
- Clarified input validation and context discovery documentation
- Enhanced output format for code review results
- Added todo list creation step in quick review commands

## [3.0.3] - 2025-01-22

### Changed
- Updated skill SKILL.md files to use consistent imperative writing style
- Shortened command descriptions to under 60 characters for better `/help` display
- Added this CHANGELOG.md for tracking plugin changes

### Fixed
- Skill "Applicable Contexts" sections renamed to "When to Use" with action-oriented bullet points

## [3.0.2] - 2025-01-21

### Changed
- Updated performance-agent model from Sonnet to Opus for enhanced analysis accuracy
- Enhanced code review documentation for comprehensive 9-agent analysis

## [3.0.1] - 2025-01-21

### Added
- Usage tracking initialization and summary generation for code reviews

## [3.0.0] - 2025-01-20

### Added
- **Synthesis Agent**: New cross-agent synthesis for detecting ripple effects and cross-cutting concerns
- **Two-Phase Review Pipeline**: Sequential thorough → gaps approach with context passing
- **Usage Tracking**: Agent invocation timing and anomaly detection
- **Quick Review Synthesis**: 3 synthesis agents for quick reviews (7 total invocations)

### Changed
- Deep review now uses 16 agent invocations (8 thorough + 4 gaps + 4 synthesis)
- Quick review now uses 7 agent invocations (4 review + 3 synthesis)
- Gaps mode agents receive Phase 1 findings to avoid duplicates
- All gaps mode agents use Sonnet for cost efficiency

### Architecture
- 9-agent modular architecture with parameterized MODE support
- Agents: compliance, bug-detection, security, performance, architecture, api-contracts, error-handling, test-coverage, synthesis
- Each agent has unique color for visual distinction in parallel execution

## [2.x.x] - Previous Versions

### Features
- Initial 8-agent architecture
- Basic thorough and quick review modes
- Node.js/TypeScript and .NET/C# language support
- Validation layer for false positive reduction
- Severity classification (Critical, Major, Minor, Suggestion)
- Consensus scoring for multi-agent flagged issues

---

## Version Locations

When releasing a new version, update:

**Required:**
- `.claude-plugin/plugin.json` - Plugin manifest
- Repository root `.claude-plugin/marketplace.json` (if applicable)

**Recommended:**
- `agents/*.md` and `agents/docs/*.md` - Agent frontmatter version field (16 files)
- `skills/*/SKILL.md` - Skill frontmatter version field (7 files)

**Verification:**
```bash
# Verify no old version remains (exclude CHANGELOG history)
grep -r "<prev>" --include="*.md" --include="*.json" | grep -v CHANGELOG

# Verify new version count (~26 expected)
grep -r "<new>" --include="*.md" --include="*.json" | grep -v CHANGELOG | wc -l
```
