---
name: reviewing-architecture-principles
description: Use when checking for architecture violations, coupling problems, or maintainability concerns during code review.
---

# Architecture Principles Review Skill

Adds scope prioritization, architecture FP adjustments, SOLID/DRY/YAGNI pattern references.

## Scope Prioritization

- Service/business logic files
- Files with "Factory", "Service", "Manager", "Handler" in names
- Files with 500+ lines
- Files with many exports

## False Positives

**Architecture-specific** - do NOT flag:
- Intentional duplication for clarity (documented)
- Abstractions planned for imminent use (documented roadmap)
- Pragmatic compromises with documented justification
- Patterns appropriate for project scale

## Pattern References

`references/solid-dry-yagni-patterns.md`
