# AI Agent Instruction File Templates

## File Structure

```
repository/
├── .ai/
│   └── AI-AGENT-INSTRUCTIONS.md    # Comprehensive coding standards
├── .github/
│   └── copilot-instructions.md     # GitHub Copilot quick reference
└── CLAUDE.md                       # Claude Code quick reference
```

## Required Headers

- **AI-AGENT-INSTRUCTIONS.md**: Must start with `# AI Agent Instructions` and state "This file contains comprehensive coding standards..." with cross-ref note to CLAUDE.md and copilot-instructions.md. Sections: Quick Reference, Coding Standards, Architecture, Testing, Documentation, Common Patterns, Anti-Patterns.
- **CLAUDE.md**: Must start with `# CLAUDE.md` and state "This file provides guidance to Claude Code..." with link to `.ai/AI-AGENT-INSTRUCTIONS.md` and sync note to copilot-instructions.md.
- **copilot-instructions.md**: Must start with `# Copilot Instructions` and state "This file provides guidance to GitHub Copilot..." with link to `../.ai/AI-AGENT-INSTRUCTIONS.md` and sync note to CLAUDE.md.

## Cross-References

All three files must cross-reference each other. Quick Reference sections must contain the same essential information across all three files.

## Detection Threshold

Flag missing AI instruction files only when signals suggest AI tool usage (e.g., `.claude/` directory, `CLAUDE.md` exists, `.github/copilot-instructions.md` exists).

## Validation Checklist

- `.ai/AI-AGENT-INSTRUCTIONS.md` exists (NOT in root)
- `CLAUDE.md` exists in root with cross-references
- `.github/copilot-instructions.md` exists with cross-references
- All cross-reference links are valid
- Quick Reference sections are in sync
