---
name: reviewing-technical-debt
description: Detects deprecated dependencies, outdated patterns, workarounds, dead code, and scalability concerns. Use when assessing technical debt, identifying dead code, or evaluating code modernization needs during code review.
---

# Technical Debt Code Review Skill

## Agent

`code-review:technical-debt-agent` (Opus) in thorough mode.

## Technical Debt Categories Checked

**Deprecated Dependencies (Critical/Major):**
- Packages with deprecation warnings
- Major version 2+ behind with breaking changes
- Discontinued or unmaintained libraries
- Dependencies with known CVEs requiring upgrade

**Outdated Patterns (Minor/Suggestion):**
- Callback patterns where async/await is standard
- Class components in modern React projects
- CommonJS in ESM-configured projects
- Legacy bundler configurations
- Sync-over-async patterns in .NET

**Workarounds and Hacks (Major/Minor):**
- HACK, WORKAROUND, XXX comments
- Temporary fixes that became permanent
- Monkey patches and runtime modifications
- Version-specific workarounds for fixed issues

**Dead Code (Minor):**
- Unused exports never imported
- Commented-out code blocks (10+ lines)
- Unreachable code paths
- Abandoned feature flags

**Scalability Concerns (Major):**
- Patterns that won't scale
- Hardcoded limits
- In-memory solutions needing externalization
- Unbounded collections

**Documentation Debt (Minor/Suggestion):**
- TODO/FIXME without issue tracking
- Stale comments describing wrong behavior
- Missing API documentation

## Auto-Validated Patterns

High-confidence patterns skip validation. Full definitions with regex patterns: `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md`.

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

`examples/example-output.md`

