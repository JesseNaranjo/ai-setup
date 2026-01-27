---
name: technical-debt-review
description: This skill should be used when the user asks to "find technical debt", "check for deprecated code", "find outdated patterns", "identify dead code", "check for workarounds", "find TODO comments", "assess code health", "code modernization review", "legacy code review", or wants to identify accumulated technical debt.
version: 3.2.0
---

# Technical Debt Code Review Skill

Identify accumulated technical debt including deprecated dependencies, outdated patterns, workarounds, dead code, scalability concerns, and documentation debt.

## Workflow

**Agent:** `code-review:technical-debt-agent` (see `orchestration-sequence.md` for model selection)

See `${CLAUDE_PLUGIN_ROOT}/shared/skill-common-workflow.md` for common procedures.

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

---

## Auto-Validated Patterns

High-confidence patterns that skip validation. For full definitions with regex patterns, see `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md`.

**Technical Debt patterns:** `deprecated_npm_package`, `todo_without_issue`, `commented_out_code`, `hack_comment`, `outdated_callback`

---

## Scope Prioritization

When reviewing directories, automatically prioritize:
- Package manifest files (`package.json`, `*.csproj`)
- Configuration files (bundler configs, tsconfig)
- Files with many TODO/FIXME comments
- Old utility files not recently modified

---

## False Positives

Apply all rules from `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md` "False Positive Rules" section.

**Technical Debt-specific additions** - do NOT flag:
- Dependencies intentionally pinned with documented reason
- Legacy patterns in explicitly deprecated modules
- TODOs that reference issue tracking (e.g., `TODO(#123)`)
- Workarounds with documented upstream bug references

---

## Example Output

See `examples/example-output.md` for samples showing:
- Deprecated dependency finding with prompt fix
- TODO without issue tracking with diff fix
- Outdated pattern with prompt fix
- Dead code with diff fix

