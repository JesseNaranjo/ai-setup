# Documentation Best Practices Reference

## README.md Required Sections

1. **Title/Description** - name, one-paragraph description, key features
2. **Installation** - prerequisites with versions, step-by-step, verification
3. **Quick Start** - minimal working example, expected output
4. **Configuration** - required env vars, config format, defaults
5. **API/CLI Reference** - commands/methods, parameters, return values
6. **Contributing** - bug reports, PR process, code style
7. **License**

**Anti-patterns**: wall of text without headings, assuming global deps, non-working examples, missing prerequisites, outdated screenshots

## Code Examples

```javascript
// GOOD: Complete, runnable, realistic values, expected output
import { createClient } from '@example/sdk';
const client = createClient({ apiKey: process.env.API_KEY });
// Output: { success: true, data: "processed input" }

// BAD: Missing imports, incomplete context
const result = await client.doSomething('input');
```

**Rules**: show imports, realistic values (not foo/bar), show expected output, handle errors, mark placeholders (`YOUR_API_KEY`)

## API Documentation

Required per endpoint: all params with types, all response codes, at least one example, error responses.

## Clarity Guidelines

- Define jargon on first use: "The ORM (Object-Relational Mapping) layer..."
- Glossary for complex projects
- Audience-appropriate depth (expandable sections for mixed audiences)

## Consistency Patterns

| Area | Rule |
|------|------|
| Terminology | One term per concept, consistent (e.g., "configuration" not "config/conf/settings") |
| Headings | Title Case or Sentence case, consistent |
| Lists | Same punctuation style throughout |
| Code | Consistent language tags, indentation |
| Links | Consistent style (inline vs reference) |

## Structure Guidelines

- Single H1 per document; TOC for documents > 3 screens
- Cross-links between related docs; relative links for internal docs
- Context in links: "See [API Reference](./api.md) for endpoint details" (not "click here")

## Versioning Documentation

- Version-specific notes when behavior differs across versions
- Deprecation notices with migration path and removal version
- Changelog: Keep a Changelog format (Added/Changed/Deprecated/Removed/Fixed/Security)
