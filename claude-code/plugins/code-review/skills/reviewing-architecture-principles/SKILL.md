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

## False Positives

**Architecture-specific additions** - do NOT flag:
- Intentional duplication for clarity (documented)
- Abstractions planned for imminent use (documented roadmap)
- Pragmatic compromises with documented justification
- Patterns appropriate for project scale

## References

`references/solid-dry-yagni-patterns.md`
