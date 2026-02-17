# Documentation Best Practices Reference

## README.md Required Sections

1. **Project Title and Description** - name, one-paragraph description, key features
2. **Installation** - prerequisites with versions, step-by-step, verification command
3. **Quick Start / Basic Usage** - minimal working example, expected output
4. **Configuration** - required env vars, config file format, defaults
5. **API/CLI Reference** - key commands/methods, parameters, return values
6. **Contributing** - bug reports, PR process, code style
7. **License**

**Anti-patterns**: wall of text without headings, installation assuming global deps, non-working examples, missing prerequisites, outdated screenshots

## Code Examples

```javascript
// GOOD: Complete, runnable with imports, realistic values, expected output
import { createClient } from '@example/sdk';
const client = createClient({ apiKey: process.env.API_KEY });
// Output: { success: true, data: "processed input" }

// BAD: Missing imports, incomplete context
const result = await client.doSomething('input');
```

**Rules**: show imports, use realistic values (not foo/bar), show expected output, handle errors, mark placeholders (`YOUR_API_KEY`)

## API Documentation

Required per endpoint: all parameters with types, all response codes, at least one example, error responses.

## Clarity Guidelines

- Define jargon on first use: "The ORM (Object-Relational Mapping) layer..."
- Maintain glossary for complex projects
- Audience-appropriate depth (expandable sections for mixed audiences)

## Consistency Patterns

| Area | Rule |
|------|------|
| Terminology | Pick one term per concept, use consistently (e.g., "configuration" not "config/conf/settings") |
| Headings | Choose Title Case or Sentence case, stick with it |
| Lists | Same punctuation style throughout |
| Code | Consistent language tags, indentation |
| Links | Consistent style (inline vs reference) |

## Structure Guidelines

- Single H1 per document; table of contents for documents > 3 screens
- Cross-links between related documents; use relative links for internal docs
- Provide context in links: "See [API Reference](./api.md) for endpoint details" (not "click here")

## Versioning Documentation

- Version-specific notes when behavior differs across versions
- Deprecation notices with migration path and removal version
- Changelog: Keep a Changelog format, group by Added/Changed/Deprecated/Removed/Fixed/Security
