---
name: reviewing-technical-debt
description: Detects deprecated dependencies, outdated patterns, workarounds, dead code, and scalability concerns. Use when assessing technical debt, identifying dead code, or evaluating code modernization needs during code review.
---

# Technical Debt Code Review Skill

Enhancement: Adds scope prioritization (dependency/config files), technical-debt-specific false positive adjustments, and detailed debt pattern references.

## Agent

`code-review:technical-debt-agent` (Opus) in thorough mode.

## Auto-Validated Patterns

High-confidence patterns skip validation. Full definitions with regex patterns: `${CLAUDE_PLUGIN_ROOT}/shared/review-validation-code.md`.

## Scope Prioritization

Prioritize:
- Package manifest files (`package.json`, `*.csproj`)
- Configuration files (bundler configs, tsconfig)
- Files with many TODO/FIXME comments
- Old utility files not recently modified

## False Positives

Apply `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md` "False Positive Rules".

**Technical Debt-specific additions** - do NOT flag:
- Dependencies intentionally pinned with documented reason
- Legacy patterns in explicitly deprecated modules
- TODOs that reference issue tracking (e.g., `TODO(#123)`)
- Workarounds with documented upstream bug references

## Detailed Debt Patterns

`references/technical-debt-patterns.md`

## Example Output

`${CLAUDE_PLUGIN_ROOT}/shared/example-output.md`

