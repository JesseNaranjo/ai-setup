# Context Discovery

Instructions for gathering review context before the main review phase.

### Step 1: Find AI Agent Instructions Files

Search for instruction files in priority order (higher priority files override lower):

```
Priority 1 (highest): CLAUDE.md in directories of files being reviewed
Priority 2: CLAUDE.md in repository root
Priority 3: .ai/AI-AGENT-INSTRUCTIONS.md
Priority 4: .github/copilot-instructions.md
```

For each file being reviewed, identify which instruction files apply (same directory or any parent directory up to repo root).

### Step 2: Detect Language Per File

Detect the programming language for EACH file being reviewed using file extension and nearest project file (`package.json` → Node.js, `*.csproj`/`*.sln`/`*.slnx` → .NET/C#).

**Handling monorepos and mixed codebases:**

- Apply language-specific checks PER FILE, not globally
- Group files by detected language when building review context

**Language override:** `--language nodejs|dotnet|react` overrides auto-detection for ALL files.

### Step 2b: Detect Frameworks

For Node.js/TypeScript files, check nearest `package.json` for `react` or `react-dom` in dependencies/devDependencies. `--language react` implies Node.js base checks + React-specific checks.

### Step 3: Find Related Test Files

For each file being reviewed, find corresponding test files based on detected language. See `${CLAUDE_PLUGIN_ROOT}/languages/nodejs.md` and `${CLAUDE_PLUGIN_ROOT}/languages/dotnet.md` for default test patterns.

**Merge user-configured patterns:** If `additional_test_patterns` is provided in `.claude/code-review.local.md`, merge with default patterns. User patterns are checked in addition to the defaults.

### Step 4: Return Discovery Results

Return a structured result with all discovered context:

```yaml
discovery_results:
  ai_instructions:
    - path: "CLAUDE.md"
      applies_to: ["src/api/handler.ts"]
  detected_languages:
    nodejs: ["src/api/handler.ts"]
    dotnet: ["Services/UserService.cs"]
  detected_frameworks:
    react: ["src/components/Button.tsx"]
  test_files:
    - source: "src/api/handler.ts"
      tests: ["src/api/__tests__/handler.test.ts"]
  language_override: null  # or "nodejs" / "react" / "dotnet" if --language was used
```

**Note:** `detected_languages` enables lazy loading of language configs - only load `languages/nodejs.md` if `detected_languages.nodejs` has files. `detected_frameworks` tracks React detection that extends base language configs.

## Error Handling

- If no AI instruction files are found, continue with empty list (review still proceeds)
- If language cannot be detected for a file, log a warning and skip language-specific checks
- If test files cannot be found, continue without test context (not a blocker)
