# Documentation Review Example Output

Sample output showing various documentation issue types and fix formats.

## Accuracy Issue (Diff Fix)

```yaml
issues:
  - title: "Incorrect function signature in API docs"
    file: "docs/api/users.md"
    line: 45
    category: "Accuracy"
    severity: "Critical"
    description: "Documentation shows createUser(name) but the actual function signature is createUser(name, email). Users following the docs will get runtime errors."
    documented_value: "createUser(name: string): User"
    actual_value: "createUser(name: string, email: string): User"
    code_location: "src/api/users.ts:23"
    fix_type: "diff"
    fix_diff: |
      - ### createUser(name)
      + ### createUser(name, email)

      - Creates a new user with the given name.
      + Creates a new user with the given name and email.

        **Parameters:**
        | Name | Type | Description |
        |------|------|-------------|
        | name | string | User's display name |
      + | email | string | User's email address |
```

## Broken Link (Diff Fix)

```yaml
issues:
  - title: "Broken internal link to API reference"
    file: "README.md"
    line: 67
    category: "Structure"
    severity: "Major"
    description: "Link points to docs/api.md but file was renamed to docs/api-reference.md"
    structure_type: "links"
    broken_target: "docs/api.md"
    fix_type: "diff"
    fix_diff: |
      - For detailed API documentation, see [API Reference](docs/api.md).
      + For detailed API documentation, see [API Reference](docs/api-reference.md).
```

## AI Instruction Standardization (Prompt Fix)

```yaml
issues:
  - title: "AI-AGENT-INSTRUCTIONS.md in wrong location"
    file: "AI-AGENT-INSTRUCTIONS.md"
    line: 1
    category: "Structure"
    severity: "Major"
    description: "AI-AGENT-INSTRUCTIONS.md exists in repository root but should be in .ai/ directory per standardization requirements."
    structure_type: "ai_instructions"
    fix_type: "prompt"
    fix_prompt: |
      Move AI-AGENT-INSTRUCTIONS.md to the standard location:
      1. Create .ai/ directory if it doesn't exist: mkdir -p .ai
      2. Move the file: mv AI-AGENT-INSTRUCTIONS.md .ai/
      3. Update the header to include the standard format with cross-references
      4. Update CLAUDE.md to reference .ai/AI-AGENT-INSTRUCTIONS.md
      5. Update or create .github/copilot-instructions.md with reference
```

## Missing Section (Prompt Fix)

```yaml
issues:
  - title: "No troubleshooting section in getting started guide"
    file: "docs/getting-started.md"
    line: 200
    category: "Completeness"
    severity: "Major"
    description: "Documentation lacks troubleshooting section. Common issues (port conflicts, database connection, missing env vars) are not addressed, leading to frequent support requests."
    missing_type: "section"
    fix_type: "prompt"
    fix_prompt: |
      Add a Troubleshooting section at the end of docs/getting-started.md:

      ## Troubleshooting

      ### Port Already in Use
      Error: `EADDRINUSE: address already in use :::3000`
      Solution: Change the port in .env or stop the process using port 3000.

      ### Database Connection Failed
      Error: `ECONNREFUSED 127.0.0.1:5432`
      Solution: Ensure PostgreSQL is running and DATABASE_URL is correct.

      ### Missing Environment Variables
      Error: `API_KEY is not defined`
      Solution: Copy .env.example to .env and fill in required values.
```

## Clarity Issue (Diff Fix)

```yaml
issues:
  - title: "Undefined acronym 'JWT'"
    file: "docs/auth.md"
    line: 12
    category: "Clarity"
    severity: "Major"
    description: "JWT is used without definition. Users unfamiliar with authentication may not know this acronym."
    affected_audience: "beginner"
    clarity_type: "jargon"
    fix_type: "diff"
    fix_diff: |
      - Authentication is handled via JWT tokens.
      + Authentication is handled via JWT (JSON Web Token) tokens.
```

## Example Error (Diff Fix)

```yaml
issues:
  - title: "Missing import in code example"
    file: "docs/quickstart.md"
    line: 34
    category: "Examples"
    severity: "Critical"
    description: "Example uses 'createClient' but doesn't show the import. Users will get 'createClient is not defined' when copying this code."
    example_type: "imports"
    language: "javascript"
    error_message: "ReferenceError: createClient is not defined"
    fix_type: "diff"
    fix_diff: |
        ```javascript
      + import { createClient } from '@example/sdk';
      +
        const client = createClient({ apiKey: 'your-key' });
        const result = await client.getData();
        ```
```

## Consistency Issue (Prompt Fix)

```yaml
issues:
  - title: "Inconsistent terminology: 'config' vs 'configuration'"
    file: "docs/"
    line: 1
    category: "Consistency"
    severity: "Minor"
    description: "Documentation uses both 'config' and 'configuration' interchangeably. 'configuration' appears 15 times, 'config' appears 8 times across docs."
    consistency_type: "terminology"
    variant_a: "config"
    variant_b: "configuration"
    recommended: "configuration"
    other_locations:
      - "docs/setup.md:23"
      - "docs/setup.md:45"
      - "docs/api.md:12"
      - "README.md:34"
    fix_type: "prompt"
    fix_prompt: |
      Standardize terminology to use 'configuration' consistently:

      1. In docs/setup.md line 23: "Create a config file" → "Create a configuration file"
      2. In docs/setup.md line 45: "config options" → "configuration options"
      3. In docs/api.md line 12: "config object" → "configuration object"
      4. In README.md line 34: "config/" → "configuration/"

      Exception: Keep 'config' in file names and code references where it matches actual names.
```

## Cross-Cutting Insight (Synthesis)

```yaml
cross_cutting_insights:
  - categories: ["Accuracy", "Examples"]
    insight: "3 code examples in docs/api.md use the old v1 API signatures while the prose describes v2 behavior. The examples should be updated to match the documented v2 API."
    related_findings:
      accuracy: ["Incorrect function signature in API docs"]
      examples: ["Deprecated API usage in example", "Wrong parameter count in example"]

  - categories: ["Clarity", "Structure"]
    insight: "The confusing authentication flow explanation (Clarity issue) is made worse by the scattered location of auth content across 3 different files. Consolidating auth documentation would address both issues."
    related_findings:
      clarity: ["Authentication flow explanation is confusing"]
      structure: ["Auth documentation split across multiple files"]
```

## Complete Report Summary

```markdown
# Documentation Review Report

**Review Type:** Deep Documentation Review (13 invocations)
**Files Reviewed:** 8
**Total Issues:** 12

## Summary by Severity

| Severity | Count |
|----------|-------|
| Critical | 2 |
| Major | 6 |
| Minor | 3 |
| Suggestion | 1 |

## Summary by Category

| Category | Count |
|----------|-------|
| Accuracy | 3 |
| Clarity | 2 |
| Completeness | 2 |
| Consistency | 2 |
| Examples | 2 |
| Structure | 1 |

## AI Instruction File Status

| File | Status |
|------|--------|
| .ai/AI-AGENT-INSTRUCTIONS.md | ⚠️ In wrong location (root) |
| CLAUDE.md | ✅ Exists |
| .github/copilot-instructions.md | ❌ Missing |

## Cross-Cutting Insights

2 cross-cutting concerns identified affecting multiple categories.

---

[Full findings below...]
```
