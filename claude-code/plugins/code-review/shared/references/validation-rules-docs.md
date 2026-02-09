# Documentation Review Validation Rules

This section defines the validation process and domain-specific patterns for documentation review commands.

## Batch Validation Process

See `${CLAUDE_PLUGIN_ROOT}/shared/validation-aggregation.md` for the common validation process (grouping, cross-cutting insight validation, batch validator prompt, false positives, verdicts).

### Validator Model Assignment

| Issue Category | Validator Model |
|----------------|-----------------|
| Accuracy | Opus |
| Clarity | Sonnet |
| Completeness | Opus |
| Consistency | Sonnet |
| Examples | Opus |
| Structure | Sonnet |

**Cross-cutting insights** (from synthesis-docs-agent) always use **Opus** for validation.

**Note for Quick Reviews:** Despite the quick review philosophy of using Sonnet where possible, cross-cutting insights still use Opus for validation because they represent novel connections between categories that require more nuanced judgment to validate. The time savings of Sonnet validation does not justify the risk of missing subtle cross-category interactions.

### Auto-Validation (Skip Validation)

Some high-confidence patterns skip validation entirely and are marked `auto_validated: true`:

**Domain-specific patterns:** See Auto-Validation Patterns (Documentation) section below.

## Aggregation Rules

See `${CLAUDE_PLUGIN_ROOT}/shared/validation-aggregation.md` for aggregation rules (remove invalid, apply downgrades, deduplicate, consensus detection).

## Auto-Validation Patterns (Documentation)

Some high-confidence patterns skip validation entirely and are marked `auto_validated: true`:

**Accuracy patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `api_signature_mismatch` | N/A (detected via code comparison) | Documented function signature differs from implementation |
| `missing_parameter_doc` | N/A (detected via param comparison) | Parameter exists in code but not documented |
| `outdated_version_reference` | N/A (detected via version comparison) | Documentation references older version than package.json/csproj |

**Clarity patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `undefined_acronym` | `\b[A-Z]{2,}\b(?!.*\([^)]+\))` | Acronym used without expansion (first occurrence) |
| `passive_voice_instruction` | N/A (detected via NLP analysis) | Instructions written in passive voice |

**Completeness patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `missing_api_doc` | N/A (detected via export comparison) | Exported function/class has no documentation |
| `empty_section` | `^#+\s+.+\n\s*\n(?=^#+)` | Section header with no content before next header |
| `missing_error_handling_doc` | N/A (detected via throw analysis) | Function throws but no error documentation |

**Consistency patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `inconsistent_terminology` | N/A (detected via term frequency analysis) | Same concept named differently across docs |
| `inconsistent_heading_style` | N/A (detected via heading pattern analysis) | Mixed title case and sentence case headings |

**Examples patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `example_syntax_error` | N/A (detected via parser) | Code example has syntax error |
| `example_undefined_import` | N/A (detected via import analysis) | Example uses undefined import |

**Structure patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `broken_internal_link` | `\[([^\]]+)\]\((?!https?://)([^)]+)\)` | Internal markdown link (check target exists) |
| `missing_ai_instruction_header` | N/A (detected via header check) | CLAUDE.md missing required header comment |
| `ai_instruction_wrong_location` | N/A (detected via path check) | AI instruction file in wrong directory |

## Category-Specific False Positive Rules (Documentation)

Each category has specific exclusions in addition to the general false positive rules in `${CLAUDE_PLUGIN_ROOT}/shared/agent-common-instructions.md`.

### Accuracy

- Intentionally simplified examples (marked as "simplified" or "basic example")
- Pseudocode clearly marked as illustrative
- Version-specific documentation with version clearly noted
- Documentation for planned/upcoming features marked as such

### Clarity

- Jargon appropriate for stated expert audience
- Acronyms defined earlier in the same document
- Industry-standard terms in domain-specific docs (e.g., "REST" in API docs)
- Intentionally terse reference documentation (vs tutorials)
- Code comments within code blocks (different standards apply)

### Completeness

- Internal/private APIs not intended for external use
- Features clearly marked as experimental/unstable
- Configuration options with sensible defaults that rarely need changing
- Platform-specific docs when project only targets one platform
- Sections that would duplicate content available elsewhere (with link)

### Consistency

- Intentional variations for emphasis or clarity
- Code/API names that must match implementation (even if inconsistent with prose style)
- Quoted text that preserves original formatting
- Version-specific sections that intentionally differ
- External content (quotes, references) with different style

### Examples

- Pseudocode clearly marked as illustrative (not runnable)
- Intentionally simplified examples with explicit notes about what's omitted
- Partial examples with "..." indicating omitted code
- Examples for older versions in clearly versioned documentation
- Examples showing error cases (intentionally incorrect code to demonstrate what not to do)
- Shell examples with placeholder values like `<your-token>`

### Structure

- Intentionally orphaned archive/historical documents
- External links to known-stable resources (official docs, RFCs)
- Heading hierarchy violations in code-generated documentation
- Alternative navigation paths that are intentional (multiple entry points)
- AI instruction files in projects that don't use AI assistants (if explicitly stated)
