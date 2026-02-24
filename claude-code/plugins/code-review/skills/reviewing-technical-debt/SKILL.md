---
name: reviewing-technical-debt
description: Detects deprecated dependencies, outdated patterns, workarounds, dead code, and scalability concerns. Use when assessing technical debt, identifying dead code, or evaluating code modernization needs during code review.
---

# Technical Debt Code Review Skill

Adds scope prioritization (dependency/config files), debt-specific FP adjustments, debt pattern references.

## Scope Prioritization

- Package manifest files (`package.json`, `*.csproj`)
- Configuration files (bundler configs, tsconfig)
- Files with many TODO/FIXME comments
- Old utility files not recently modified

## False Positives

**Debt-specific** - do NOT flag:
- Dependencies intentionally pinned with documented reason
- Legacy patterns in explicitly deprecated modules
- TODOs referencing issue tracking (e.g., `TODO(#123)`)
- Workarounds with documented upstream bug references

## Pattern References

`references/technical-debt-patterns.md`
