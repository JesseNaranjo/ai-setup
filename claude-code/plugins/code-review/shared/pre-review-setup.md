# Pre-Review Setup

## Section 1: Settings Loader

Load and apply project-specific settings from `.claude/code-review.local.md`.

### Loading Process

Before starting the review, check for settings:

#### 1. Check for Settings File

Look for `.claude/code-review.local.md` in the project root.

If the file does not exist, use default settings and continue.

#### 2. Parse YAML Frontmatter

If the file exists, parse the YAML frontmatter to extract settings.

Fields: `enabled`, `output_dir`, `skip_agents`, `min_severity`, `language` (empty = auto-detect), `additional_test_patterns`.

#### 3. Check Enabled Flag

If `enabled: false`, stop and inform the user: "Code review plugin is disabled for this project. Edit .claude/code-review.local.md to enable."

#### 4. Apply Settings

Apply settings to the review process:

| Setting | How to Apply |
|---------|--------------|
| `output_dir` | Prepend to default output filename (unless --output-file overrides) |
| `skip_agents` | Exclude listed agents from the review phases |
| `min_severity` | Filter output to only show issues at or above this severity |
| `language` | Use as language override (unless --language flag overrides) |
| `additional_test_patterns` | Merge with default test patterns when finding related tests |

#### 5. Read Project Instructions

Read the markdown body (after the YAML frontmatter) and pass it to all review agents as additional context under "Project-Specific Instructions".

Command-line flags override corresponding settings; `--prompt` appends to project instructions.

## Section 2: Context Discovery

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

Return a structured `discovery_results` with fields: `ai_instructions` (path + applies_to), `detected_languages` (language → file list), `detected_frameworks` (framework → file list), `test_files` (source → tests), `language_override` (from --language flag or null).

**Note:** `detected_languages` enables lazy loading of language configs — only load `languages/nodejs.md` if `detected_languages.nodejs` has files. `detected_frameworks` tracks React detection that extends base language configs.

If language cannot be detected for a file, log a warning and skip language-specific checks.
