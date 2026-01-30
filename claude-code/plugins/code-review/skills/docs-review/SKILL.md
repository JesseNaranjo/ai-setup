---
name: docs-review
description: This skill should be used when the user asks to "review documentation", "check docs", "audit README", "check CLAUDE.md", "verify AI instructions", "standardize docs", "review markdown", "check docs accuracy", or mentions reviewing project documentation for quality issues.
version: 3.3.0
---

# Documentation Review Skill

Identify documentation issues including accuracy, clarity, completeness, consistency, examples, and structure through targeted documentation-focused review.

## Workflow

**Commands:**
- `/deep-docs-review` - Full 13-invocation pipeline for comprehensive review
- `/quick-docs-review` - Quick 7-invocation pipeline for critical issues

1. **Scope**: Review files specified by user or discover all documentation
2. **Context**: Detect project type, gather version info and code references
3. **Launch**: Invoke documentation agents with appropriate MODE
4. **Validate**: Issues validated per validation-rules.md
5. **Report**: Output findings using YAML schema with fix_type

## Documentation Categories Checked

**Accuracy (Critical):**
- Code-documentation synchronization
- API signature correctness
- Version number accuracy
- CLI command validity
- Example correctness

**Clarity (Major):**
- Unexplained jargon and acronyms
- Ambiguous explanations
- Audience appropriateness
- Readability issues

**Completeness (Major):**
- Missing standard sections
- Undocumented features/APIs
- Incomplete setup instructions
- Missing troubleshooting

**Consistency (Minor):**
- Terminology variations
- Formatting inconsistencies
- Voice and tone mismatches
- Naming convention violations

**Examples (Critical):**
- Syntax errors in code blocks
- Missing imports
- Incorrect API usage
- Outdated examples

**Structure (Major):**
- Broken links (internal and external)
- Heading hierarchy issues
- Navigation problems
- **AI instruction file standardization**

---

## AI Instruction File Standardization

This skill includes checks for AI agent instruction file compliance:

**Required Files:**
- `/.ai/AI-AGENT-INSTRUCTIONS.md` - Comprehensive coding standards
- `/CLAUDE.md` - Claude Code quick reference
- `/.github/copilot-instructions.md` - GitHub Copilot quick reference

**Standardization Rules:**
1. AI-AGENT-INSTRUCTIONS.md must be in `/.ai/` (not root)
2. CLAUDE.md must reference `.ai/AI-AGENT-INSTRUCTIONS.md` in header
3. copilot-instructions.md must exist with correct header
4. Cross-references between all three files must be valid

See `references/ai-instruction-templates.md` for required headers.

---

## Scope Prioritization

When reviewing directories, automatically prioritize:
- README.md (project entry point)
- CLAUDE.md (AI assistant instructions)
- docs/getting-started.md or similar
- API documentation
- AI instruction files

---

## False Positives

Apply all rules from `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md`.

**Documentation-specific additions** - do NOT flag:
- Intentionally simplified examples (marked as such)
- Placeholder values (`your-api-key`, `example.com`)
- Version-specific docs with version clearly noted
- Pseudocode marked as illustrative

---

## References

For detailed patterns and templates:
- **`references/ai-instruction-templates.md`** - Templates for AI Agent Instructions files
- **`references/documentation-best-practices.md`** - Documentation quality patterns

---

## Example Output

See `examples/example-output.md` for samples showing:
- Accuracy issue with diff fix
- Broken link with diff fix
- AI instruction standardization issue with prompt fix
- Missing section with prompt fix
