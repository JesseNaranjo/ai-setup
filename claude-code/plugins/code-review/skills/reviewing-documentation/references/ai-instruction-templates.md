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

### .ai/AI-AGENT-INSTRUCTIONS.md

Comprehensive coding standards. Required header:

```markdown
# AI Agent Instructions

This file contains comprehensive coding standards, patterns, and conventions for this repository. AI coding assistants (Claude Code, GitHub Copilot, etc.) MUST follow these guidelines.

**This is the detailed reference document for this repository.**

> **Note:** You MUST keep the Quick Reference in sync with [CLAUDE.md](../CLAUDE.md) and [.github/copilot-instructions.md](../.github/copilot-instructions.md).
```

Sections: Quick Reference, Coding Standards, Architecture, Testing, Documentation, Common Patterns, Anti-Patterns

### CLAUDE.md

Quick reference for Claude Code:

```markdown
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**For comprehensive coding standards, see [.ai/AI-AGENT-INSTRUCTIONS.md](.ai/AI-AGENT-INSTRUCTIONS.md).**

> **Note:** You MUST keep this file in sync with [.github/copilot-instructions.md](.github/copilot-instructions.md).
```

### .github/copilot-instructions.md

Quick reference for GitHub Copilot:

```markdown
# Copilot Instructions

This file provides guidance to GitHub Copilot when working with code in this repository.

**For comprehensive coding standards, see [.ai/AI-AGENT-INSTRUCTIONS.md](../.ai/AI-AGENT-INSTRUCTIONS.md).**

> **Note:** You MUST keep this file in sync with [CLAUDE.md](../CLAUDE.md).
```

## Cross-Reference Requirements

| From | To | Link |
|------|-----|------|
| CLAUDE.md | AI-AGENT-INSTRUCTIONS.md | `[.ai/AI-AGENT-INSTRUCTIONS.md](.ai/AI-AGENT-INSTRUCTIONS.md)` |
| CLAUDE.md | copilot-instructions.md | `[.github/copilot-instructions.md](.github/copilot-instructions.md)` |
| copilot-instructions.md | AI-AGENT-INSTRUCTIONS.md | `[.ai/AI-AGENT-INSTRUCTIONS.md](../.ai/AI-AGENT-INSTRUCTIONS.md)` |
| copilot-instructions.md | CLAUDE.md | `[CLAUDE.md](../CLAUDE.md)` |
| AI-AGENT-INSTRUCTIONS.md | CLAUDE.md | `[CLAUDE.md](../CLAUDE.md)` |
| AI-AGENT-INSTRUCTIONS.md | copilot-instructions.md | `[.github/copilot-instructions.md](../.github/copilot-instructions.md)` |

Quick Reference sections must contain the same essential information across all three files.

## Validation Checklist

- `.ai/AI-AGENT-INSTRUCTIONS.md` exists (NOT in root)
- `CLAUDE.md` exists in root with cross-references
- `.github/copilot-instructions.md` exists with cross-references
- All cross-reference links are valid
- Quick Reference sections are in sync
