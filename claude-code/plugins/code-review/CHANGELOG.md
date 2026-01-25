# Changelog

All notable changes to the Code Review Plugin are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
- `agents/*.md` - Agent frontmatter version field (9 files)
- `skills/*/SKILL.md` - Skill frontmatter version field (4 files)

**Verification:**
```bash
# Verify no old version remains (exclude CHANGELOG history)
grep -r "<prev>" --include="*.md" --include="*.json" | grep -v CHANGELOG

# Verify new version count (~17 expected)
grep -r "<new>" --include="*.md" --include="*.json" | grep -v CHANGELOG | wc -l
```
