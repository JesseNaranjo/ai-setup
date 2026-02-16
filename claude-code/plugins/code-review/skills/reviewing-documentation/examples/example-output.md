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
