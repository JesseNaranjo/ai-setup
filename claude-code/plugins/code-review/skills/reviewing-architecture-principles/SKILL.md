---
name: reviewing-architecture-principles
description: Detects coupling problems, SOLID/DRY/YAGNI violations, anti-patterns, and file organization issues. Use when user mentions "check SOLID principles", "review SOLID", "find SOLID violations", "check DRY", "find code duplication", "find duplicate code", "check YAGNI", "find over-engineering", "check SoC", "separation of concerns", "mixed concerns", "file organization", "consolidate files", or "architecture principles review".
---

# Architecture Principles Review Skill

Identify SOLID, DRY, YAGNI, and SoC violations through targeted design-principles-focused code review.

## Agent

`code-review:architecture-agent` (Opus)

Uses thorough mode with focus areas below.

## SOLID Principles Checked

**Single Responsibility (Major):**
- Classes/modules with multiple reasons to change
- Files with 500+ lines doing unrelated things
- Functions with 10+ parameters
- Mixed concerns (UI + business logic + data access)

**Open/Closed (Minor):**
- Code requiring modification to add new cases
- Switch statements on type/kind fields
- Hardcoded special cases

**Liskov Substitution (Major):**
- Subclasses throwing NotImplementedException
- Overridden methods changing behavior contracts
- Type checks on subclasses (instanceof chains)

**Interface Segregation (Minor):**
- Interfaces with 10+ methods
- Implementations with empty/throw methods
- Clients forced to depend on unused methods

**Dependency Inversion (Major):**
- Direct instantiation of dependencies
- Concrete class references instead of interfaces
- Circular dependencies between modules

---

## DRY Violations Checked

**Code Duplication (Minor):**
- Duplicated blocks >10 lines with >80% similarity
- Copy-pasted logic with minor variations
- Same validation logic in multiple handlers

**Repeated Configuration (Minor):**
- Hardcoded values that appear 3+ times
- Magic numbers/strings without constants
- Duplicate configuration across files

**Similar Utilities (Suggestion):**
- Multiple utility functions doing similar things
- Date/string formatting scattered across modules
- Redundant helper functions

---

## YAGNI Violations Checked

**Unused Abstractions (Suggestion):**
- Interfaces with single implementation
- Abstract classes never extended
- Factory patterns with one product type

**Over-Engineering (Suggestion):**
- Generic parameters never used
- Options/config that are always defaults
- Plugin systems with no plugins

**Speculative Generality (Suggestion):**
- Parameters that are always the same value
- Switch cases that are never hit
- Dead branches in configuration

---

## SoC Violations Checked

**Mixed Concerns (Major):**
- UI logic + business logic + data access in single file
- Cross-cutting concerns scattered (logging, auth, caching)
- Configuration mixed with implementation
- Test utilities mixed with production code

**Layer Violations (Major):**
- Presentation layer accessing data layer directly
- Business logic in controllers/handlers
- Database queries in UI components

---

## File Organization Checked

**Fragmentation Issues (Minor):**
- Related code scattered across many small files
- Single-use helpers in separate files (should be colocated)
- Related types/interfaces in different directories

**Consolidation Opportunities (Suggestion):**
- Multiple <20-line utility files that belong together
- Overly granular module structure
- Evaluate consolidation conservatively (prefer cohesion over fewer files)

---

## Scope Prioritization

When reviewing directories, automatically prioritize:
- Service/business logic files
- Files with "Factory", "Service", "Manager", "Handler" in names
- Files with high line counts (500+)
- Files with many exports

---

## Auto-Validated Patterns

High-confidence patterns that skip validation. For full definitions, see `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md`.

---

## False Positives

Apply all rules from `${CLAUDE_PLUGIN_ROOT}/shared/agent-common-instructions.md` "False Positive Rules" section.

**Architecture-specific additions** - do NOT flag:
- Intentional duplication for clarity (documented)
- Abstractions planned for imminent use (documented roadmap)
- Pragmatic compromises with documented justification
- Patterns appropriate for project scale

---

## Example Output

See `examples/example-output.md` for a sample showing:
- SOLID violation with prompt fix
- DRY violation with diff fix
- YAGNI violation with prompt fix
- SoC violation with prompt fix

---

## References

For detailed patterns and detection heuristics, see `references/solid-dry-yagni-patterns.md`.
