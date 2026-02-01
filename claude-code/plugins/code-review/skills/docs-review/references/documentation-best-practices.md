# Documentation Best Practices Reference

Detailed patterns and guidelines for high-quality documentation.

## Contents

- [README.md Best Practices](#readmemd-best-practices)
  - [Required Sections (Priority Order)](#required-sections-priority-order)
  - [README Anti-Patterns](#readme-anti-patterns)
- [Code Examples Best Practices](#code-examples-best-practices)
  - [Complete Examples](#complete-examples)
  - [Example Conventions](#example-conventions)
- [API Documentation Best Practices](#api-documentation-best-practices)
  - [Endpoint Documentation Template](#endpoint-documentation-template)
  - [API Documentation Requirements](#api-documentation-requirements)
- [Clarity Guidelines](#clarity-guidelines)
  - [Jargon Handling](#jargon-handling)
  - [Audience Awareness](#audience-awareness)
- [Consistency Patterns](#consistency-patterns)
  - [Terminology Consistency](#terminology-consistency)
  - [Formatting Consistency](#formatting-consistency)
- [Structure Guidelines](#structure-guidelines)
  - [Heading Hierarchy](#heading-hierarchy)
  - [Navigation](#navigation)
  - [Link Best Practices](#link-best-practices)
- [Versioning Documentation](#versioning-documentation)
  - [Version-Specific Content](#version-specific-content)
  - [Deprecation Notices](#deprecation-notices)
  - [Changelog Best Practices](#changelog-best-practices)

## README.md Best Practices

### Required Sections (Priority Order)

1. **Project Title and Description**
   - Clear, concise project name
   - One-paragraph description of what it does
   - Key features/benefits

2. **Installation**
   - Prerequisites with versions
   - Step-by-step installation
   - Verification command

3. **Quick Start / Basic Usage**
   - Minimal working example
   - Common use case
   - Expected output

4. **Configuration** (if applicable)
   - Required environment variables
   - Configuration file format
   - Default values

5. **API/CLI Reference** (or link to docs)
   - Key commands/methods
   - Parameters and options
   - Return values/output

6. **Contributing**
   - How to report bugs
   - How to submit PRs
   - Code style requirements

7. **License**
   - License type
   - Link to full license file

### README Anti-Patterns

- Wall of text without headings
- Installation assuming global dependencies exist
- Examples that don't actually work
- Missing prerequisites
- Outdated screenshots or output examples

## Code Examples Best Practices

### Complete Examples

```javascript
// ✅ Good: Complete, runnable example
import { createClient } from '@example/sdk';

const client = createClient({
  apiKey: process.env.API_KEY,
});

const result = await client.doSomething('input');
console.log(result);
// Output: { success: true, data: "processed input" }
```

```javascript
// ❌ Bad: Missing imports, incomplete
const result = await client.doSomething('input');
```

### Example Conventions

1. **Show imports** - Always include import/require statements
2. **Use realistic values** - Avoid `foo`, `bar`, `test`
3. **Show expected output** - Comment with expected results
4. **Handle errors** - Show error handling for operations that can fail
5. **Mark placeholders** - Use clear placeholder names like `YOUR_API_KEY`

## API Documentation Best Practices

### Endpoint Documentation Template

```markdown
## GET /users/:id

Retrieves a user by their unique identifier.

### Parameters

| Name | Type | Location | Required | Description |
|------|------|----------|----------|-------------|
| id | string | path | Yes | User's unique identifier |
| fields | string | query | No | Comma-separated list of fields to return |

### Response

**200 OK**
```json
{
  "id": "usr_123",
  "name": "John Doe",
  "email": "john@example.com"
}
```

**404 Not Found**
```json
{
  "error": "User not found",
  "code": "USER_NOT_FOUND"
}
```

### Example

```bash
curl -X GET "https://api.example.com/users/usr_123" \
  -H "Authorization: Bearer YOUR_TOKEN"
```
```

### API Documentation Requirements

- All endpoints documented
- All parameters listed with types
- All response codes documented
- At least one example per endpoint
- Error responses included

## Clarity Guidelines

### Jargon Handling

**Define on first use:**
```markdown
<!-- ✅ Good -->
The ORM (Object-Relational Mapping) layer handles database operations.

<!-- ❌ Bad -->
The ORM handles database operations.
```

**Maintain glossary for complex projects:**
```markdown
## Glossary

| Term | Definition |
|------|------------|
| ORM | Object-Relational Mapping - a technique for converting data between type systems |
| DTO | Data Transfer Object - an object that carries data between processes |
```

### Audience Awareness

**Beginner-friendly:**
- Explain prerequisites
- Don't assume tool familiarity
- Provide context for "why"

**Expert-focused:**
- Skip basics
- Focus on specifics
- Provide advanced options

**Mixed audience:**
- Use expandable sections for details
- Provide both quick start and deep dive
- Link to prerequisite learning resources

## Consistency Patterns

### Terminology Consistency

Create and follow a style guide:

| Preferred | Avoid |
|-----------|-------|
| configuration | config, conf, settings |
| API key | apiKey, api-key, API_KEY (in prose) |
| Node.js | NodeJS, node, Node |

### Formatting Consistency

- **Headings**: Choose Title Case or Sentence case, stick with it
- **Lists**: Use same punctuation style throughout
- **Code**: Consistent language tags, indentation
- **Links**: Consistent style (inline vs reference)

## Structure Guidelines

### Heading Hierarchy

```markdown
# Document Title (H1 - only one per document)

## Major Section (H2)

### Subsection (H3)

#### Detail (H4 - use sparingly)
```

### Navigation

- Table of contents for documents > 3 screens
- Cross-links between related documents
- Breadcrumbs or parent links in nested docs
- Clear entry points (what to read first)

### Link Best Practices

- Use relative links for internal docs
- Verify all links work
- Provide context: "See [API Reference](./api.md) for endpoint details"
- Avoid "click here" link text

## Versioning Documentation

### Version-Specific Content

```markdown
## Configuration

> **Note:** This applies to v2.x. For v1.x, see [Legacy Configuration](./v1/config.md).
```

### Deprecation Notices

```markdown
> ⚠️ **Deprecated:** `oldMethod()` is deprecated and will be removed in v3.0.
> Use `newMethod()` instead. See [Migration Guide](./migration.md).
```

### Changelog Best Practices

Follow [Keep a Changelog](https://keepachangelog.com/):
- Group by: Added, Changed, Deprecated, Removed, Fixed, Security
- Most recent version first
- Include dates
- Link to relevant PRs/issues
