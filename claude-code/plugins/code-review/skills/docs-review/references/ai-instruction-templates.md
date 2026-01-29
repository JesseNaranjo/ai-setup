# AI Agent Instruction File Templates

Required header templates and structure for AI agent instruction files.

## File Structure Overview

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

This is the **comprehensive coding standards document** that contains all detailed patterns, conventions, and rules for the repository.

**Required Header:**

```markdown
# AI Agent Instructions

This file contains comprehensive coding standards, patterns, and conventions for this repository. AI coding assistants (Claude Code, GitHub Copilot, etc.) MUST follow these guidelines.

**This is the detailed reference document for this repository.**

> **Note:** You MUST keep the Quick Reference in sync with [CLAUDE.md](../CLAUDE.md) and [.github/copilot-instructions.md](../.github/copilot-instructions.md).

## Table of Contents

- [Quick Reference](#quick-reference)
- [Coding Standards](#coding-standards)
- [Architecture](#architecture)
- [Testing](#testing)
- [Documentation](#documentation)

---

## Quick Reference

<!-- Summary of most important rules - keep in sync with CLAUDE.md and copilot-instructions.md -->
```

**Recommended Sections:**
- Quick Reference (summary for other files to reference)
- Coding Standards (formatting, naming, style)
- Architecture (patterns, structure, dependencies)
- Testing (requirements, patterns, coverage)
- Documentation (standards for docs and comments)
- Common Patterns (reusable code patterns)
- Anti-Patterns (what to avoid)

### CLAUDE.md

This is the **quick reference for Claude Code** that lives in the repository root.

**Required Header:**

```markdown
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**For comprehensive coding standards, patterns, and conventions, see [.ai/AI-AGENT-INSTRUCTIONS.md](.ai/AI-AGENT-INSTRUCTIONS.md).**

> **Note:** You MUST keep this file in sync with [.github/copilot-instructions.md](.github/copilot-instructions.md).

## Quick Reference

<!-- Keep in sync with .ai/AI-AGENT-INSTRUCTIONS.md Quick Reference section -->
```

**Recommended Content:**
- Repository overview (brief)
- Key commands (build, test, lint)
- Critical rules (most important guidelines)
- Link to detailed instructions

### .github/copilot-instructions.md

This is the **quick reference for GitHub Copilot**.

**Required Header:**

```markdown
# Copilot Instructions

This file provides guidance to GitHub Copilot when working with code in this repository.

**For comprehensive coding standards, patterns, and conventions, see [.ai/AI-AGENT-INSTRUCTIONS.md](../.ai/AI-AGENT-INSTRUCTIONS.md).**

> **Note:** You MUST keep this file in sync with [CLAUDE.md](../CLAUDE.md).

## Quick Reference

<!-- Keep in sync with .ai/AI-AGENT-INSTRUCTIONS.md Quick Reference section -->
```

**Recommended Content:**
- Same structure as CLAUDE.md
- Copilot-specific tips (if any)
- Link to detailed instructions

## Cross-Reference Requirements

### Valid Link Patterns

From each file, links to others must use correct relative paths:

| From | To | Link |
|------|-----|------|
| CLAUDE.md | AI-AGENT-INSTRUCTIONS.md | `[.ai/AI-AGENT-INSTRUCTIONS.md](.ai/AI-AGENT-INSTRUCTIONS.md)` |
| CLAUDE.md | copilot-instructions.md | `[.github/copilot-instructions.md](.github/copilot-instructions.md)` |
| copilot-instructions.md | AI-AGENT-INSTRUCTIONS.md | `[.ai/AI-AGENT-INSTRUCTIONS.md](../.ai/AI-AGENT-INSTRUCTIONS.md)` |
| copilot-instructions.md | CLAUDE.md | `[CLAUDE.md](../CLAUDE.md)` |
| AI-AGENT-INSTRUCTIONS.md | CLAUDE.md | `[CLAUDE.md](../CLAUDE.md)` |
| AI-AGENT-INSTRUCTIONS.md | copilot-instructions.md | `[.github/copilot-instructions.md](../.github/copilot-instructions.md)` |

### Sync Requirements

The Quick Reference section should contain the same essential information across all three files:

1. **Critical rules** - The most important guidelines
2. **Key commands** - Build, test, lint commands
3. **Naming conventions** - Variable, function, file naming
4. **Architecture summary** - Key patterns in use

## Common Issues

### Issue: AI-AGENT-INSTRUCTIONS.md in Root

**Problem:** File exists at `/AI-AGENT-INSTRUCTIONS.md` instead of `/.ai/AI-AGENT-INSTRUCTIONS.md`

**Fix:**
```bash
mkdir -p .ai
mv AI-AGENT-INSTRUCTIONS.md .ai/
# Update references in CLAUDE.md and copilot-instructions.md
```

### Issue: Missing Cross-References

**Problem:** CLAUDE.md doesn't reference .ai/AI-AGENT-INSTRUCTIONS.md

**Fix:** Add the required header with link to comprehensive instructions.

### Issue: Broken Relative Links

**Problem:** Link uses wrong relative path (e.g., `../AI-AGENT-INSTRUCTIONS.md` instead of `../.ai/AI-AGENT-INSTRUCTIONS.md`)

**Fix:** Correct the relative path based on the file's location.

### Issue: Missing copilot-instructions.md

**Problem:** `.github/copilot-instructions.md` doesn't exist

**Fix:** Create the file with the required header template.

## Validation Checklist

- [ ] `.ai/AI-AGENT-INSTRUCTIONS.md` exists
- [ ] `AI-AGENT-INSTRUCTIONS.md` does NOT exist in root
- [ ] `CLAUDE.md` exists in root
- [ ] `CLAUDE.md` references `.ai/AI-AGENT-INSTRUCTIONS.md`
- [ ] `CLAUDE.md` references `.github/copilot-instructions.md`
- [ ] `.github/copilot-instructions.md` exists
- [ ] `copilot-instructions.md` references `.ai/AI-AGENT-INSTRUCTIONS.md`
- [ ] `copilot-instructions.md` references `CLAUDE.md`
- [ ] All cross-reference links are valid
- [ ] Quick Reference sections are in sync
