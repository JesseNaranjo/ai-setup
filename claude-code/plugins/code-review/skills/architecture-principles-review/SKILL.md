---
name: architecture-principles-review
description: This skill should be used when the user asks to "check SOLID principles", "review SOLID", "find SOLID violations", "check DRY", "find code duplication", "find duplicate code", "check YAGNI", "find over-engineering", "architecture principles review", or wants to verify code follows software design principles.
version: 3.2.2
---

# Architecture Principles Review Skill

Identify SOLID, DRY, and YAGNI violations through targeted design-principles-focused code review.

## Workflow

**Agent:** `code-review:architecture-agent` (Opus - comprehensive architectural analysis)

1. **Scope**: Review files specified by user or staged changes (`git diff --cached`)
2. **Context**: Detect project type (Node.js via `package.json`, .NET via `*.csproj`/`*.sln`)
3. **Launch**: Invoke architecture-agent with MODE=thorough, pass skill focus areas below
4. **Validate**: Issues auto-validated if matching patterns in validation-rules.md; others validated by Sonnet
5. **Report**: Output findings using YAML schema with fix_type (diff for ≤10 line single-location fixes, prompt for complex/multi-location)

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

## Scope Prioritization

When reviewing directories, automatically prioritize:
- Service/business logic files
- Files with "Factory", "Service", "Manager", "Handler" in names
- Files with high line counts (500+)
- Files with many exports

---

## False Positives

Apply all rules from `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md` "False Positive Rules" section.

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

---

## References

For detailed patterns and detection heuristics, see `references/solid-dry-yagni-patterns.md`.
