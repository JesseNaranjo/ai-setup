# Context Discovery

Instructions for gathering review context before the main review phase.

## Agent Instructions

You are a context discovery agent. Your job is to efficiently gather all information needed for the code review agents to perform their analysis.

### Step 1: Find AI Agent Instructions Files

Search for instruction files in priority order (higher priority files override lower):

```
Priority 1 (highest): CLAUDE.md in directories of files being reviewed
Priority 2: CLAUDE.md in repository root
Priority 3: .ai/AI-AGENT-INSTRUCTIONS.md
Priority 4: .github/copilot-instructions.md
```

Use glob patterns to find these files:
- `**/CLAUDE.md`
- `.ai/AI-AGENT-INSTRUCTIONS.md`
- `.github/copilot-instructions.md`

For each file being reviewed, identify which instruction files apply (same directory or any parent directory up to repo root).

### Step 2: Detect Language Per File

Detect the programming language for EACH file being reviewed. This enables language-specific checks.

**Detection by file extension:**

| Extension(s) | Language |
|--------------|----------|
| `.ts`, `.tsx`, `.js`, `.jsx`, `.mjs`, `.cjs` | Node.js/TypeScript |
| `.cs` | .NET/C# |
| `.py` | Python (future support) |

**Detection by nearest project file:**

If extension is ambiguous or for additional validation, walk up directories from the file to find:
- `package.json` → Node.js/TypeScript
- `*.csproj`, `*.sln`, `*.slnx` → .NET/C#

Use the nearest project file to determine the language context.

**Handling monorepos and mixed codebases:**

- Apply language-specific checks PER FILE, not globally
- Group files by detected language when building review context
- Report language groupings to review agents:
  ```
  Node.js/TypeScript files: [src/api/handler.ts, src/utils/helpers.ts]
  .NET/C# files: [Services/UserService.cs, Controllers/ApiController.cs]
  ```

**Language override flag:**

If `--language` argument is provided:
- `--language nodejs` → Force Node.js/TypeScript checks for ALL files
- `--language dotnet` → Force .NET/C# checks for ALL files

The override takes precedence over auto-detection.

### Step 2b: Detect Frameworks

For files detected as Node.js/TypeScript, additionally check for React framework:

**React detection:**
- Read nearest `package.json` to the file
- If `dependencies` or `devDependencies` contains `react` or `react-dom`: mark file as React
- Store in `detected_frameworks.react` for language config loading

**Framework override:**

If `--language react` is provided:
- Force React checks for all Node.js/TypeScript files
- This implies Node.js base checks + React-specific checks

### Step 3: Find Related Test Files

For each file being reviewed, find corresponding test files based on detected language.

**Merge user-configured patterns:**

If `additional_test_patterns` is provided in the settings file (`.claude/code-review.local.md`), merge these patterns with the default patterns below. User patterns are checked in addition to the defaults.

**Node.js/TypeScript test patterns** (defaults from `languages/nodejs.md`):
- `*.test.ts`, `*.spec.ts`
- `*.test.js`, `*.spec.js`
- `*-test.js`, `*-spec.js`
- Files in `__tests__/` directories
- Files in `tests/` directories
- **Plus any patterns from `additional_test_patterns` setting**

Search strategy:
1. Same directory as source file
2. Parallel `__tests__/` directory
3. Parallel `tests/` directory
4. Match by filename (e.g., `utils.ts` → `utils.test.ts`)

**/.NET/C# test patterns** (defaults from `languages/dotnet.md`):
- `*.Tests.cs`
- Files in `*Tests/` projects
- Files in `*.UnitTests/` projects
- Files in `*.IntegrationTests/` projects
- Files in `tests/` directories
- **Plus any patterns from `additional_test_patterns` setting**

Search strategy:
1. Test project with matching name (e.g., `Services` → `Services.Tests`)
2. File with `.Tests.cs` suffix
3. Test file with matching class name

### Step 4: Return Discovery Results

Return a structured result with all discovered context:

```yaml
discovery_results:
  ai_instructions:
    - path: "CLAUDE.md"
      applies_to: ["src/api/handler.ts", "src/utils/helpers.ts"]
    - path: "src/services/CLAUDE.md"
      applies_to: ["src/services/UserService.ts"]

  files_by_language:
    nodejs:
      - "src/api/handler.ts"
      - "src/utils/helpers.ts"
    dotnet:
      - "Services/UserService.cs"

  detected_languages:
    nodejs: ["src/api/handler.ts", "src/utils/helpers.ts"]
    dotnet: ["Services/UserService.cs"]

  detected_frameworks:
    react: ["src/components/Button.tsx", "src/hooks/useAuth.ts"]

  test_files:
    - source: "src/api/handler.ts"
      tests: ["src/api/__tests__/handler.test.ts"]
    - source: "Services/UserService.cs"
      tests: ["Services.Tests/UserServiceTests.cs"]

  language_override: null  # or "nodejs" / "react" / "dotnet" if --language was used
```

**Note:** The `detected_languages` field maps each detected language to its file list. This enables lazy loading of language configs - only load `languages/nodejs.md` if `detected_languages.nodejs` has files, and only load `languages/dotnet.md` if `detected_languages.dotnet` has files. The `detected_frameworks` field tracks framework-level detection (React) that extends base language configs.

## Error Handling

- If no AI instruction files are found, continue with empty list (review still proceeds)
- If language cannot be detected for a file, log a warning and skip language-specific checks
- If test files cannot be found, continue without test context (not a blocker)
