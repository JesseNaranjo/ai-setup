---
name: reviewing-architecture-principles
description: Detects coupling problems, SOLID/DRY/YAGNI/SoC violations, anti-patterns, layer violations, and file organization issues. Use when checking for architecture violations, coupling problems, or maintainability concerns during code review.
---

# Architecture Principles Review Skill

Enhancement: Adds scope prioritization (service/business logic files, high-line-count files), architecture-specific false positive adjustments, and detailed SOLID/DRY/YAGNI pattern references.

## Agent

`code-review:architecture-agent` (Opus) in thorough mode.

## Scope Prioritization

Prioritize:
- Service/business logic files
- Files with "Factory", "Service", "Manager", "Handler" in names
- Files with high line counts (500+)
- Files with many exports

## Auto-Validated Patterns

High-confidence patterns skip validation. Full definitions: `${CLAUDE_PLUGIN_ROOT}/shared/review-validation-code.md`.

## False Positives

Apply `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md` "False Positive Rules".

**Architecture-specific additions** - do NOT flag:
- Intentional duplication for clarity (documented)
- Abstractions planned for imminent use (documented roadmap)
- Pragmatic compromises with documented justification
- Patterns appropriate for project scale

## Example Output

`${CLAUDE_PLUGIN_ROOT}/shared/example-output.md`

## References

`references/solid-dry-yagni-patterns.md`
