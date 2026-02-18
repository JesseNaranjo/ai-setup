---
name: reviewing-documentation
description: Reviews documentation for accuracy, completeness, clarity, consistency, code example validity, and structural organization. Use when reviewing documentation quality, auditing README or CLAUDE.md files, or standardizing AI instruction files.
---

# Documentation Review Skill

## Documentation Categories

6 categories: Accuracy (Critical), Clarity (Major), Completeness (Major), Consistency (Minor), Examples (Critical), Structure (Major). See docs agent files for checklists.

## AI Instruction File Standardization

Checks placement and cross-references for `.ai/AI-AGENT-INSTRUCTIONS.md`, `CLAUDE.md`, `.github/copilot-instructions.md`. See `references/ai-instruction-templates.md`.

## Scope Prioritization

- README.md (project entry point)
- CLAUDE.md (AI assistant instructions)
- docs/getting-started.md or similar
- API documentation
- AI instruction files

## False Positives

**Documentation-specific** - do NOT flag:
- Intentionally simplified examples (marked as such)
- Placeholder values (`your-api-key`, `example.com`)
- Version-specific docs with version clearly noted
- Pseudocode marked as illustrative

## References

- `references/ai-instruction-templates.md` - AI instruction templates
- `references/documentation-best-practices.md` - Documentation quality patterns
