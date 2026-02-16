# Pre-Review Setup

## Settings

Load from `.claude/code-review.local.md` in project root. If missing, use defaults.

| Field | Type | Default | Application |
|-------|------|---------|-------------|
| `enabled` | boolean | true | If false, stop with error |
| `output_dir` | string | "." | Prepend to output filename (unless --output-file) |
| `skip_agents` | array | [] | Exclude agents from review |
| `min_severity` | string | "suggestion" | Filter to issues at or above |
| `language` | string | "" | Language override (empty = auto-detect) |
| `additional_test_patterns` | array | [] | Merge with default test patterns |

Markdown body → "Project-Specific Instructions" for all agents. `--prompt` appends.
Override precedence: command-line flags > settings file > defaults.

## Context Discovery

### AI Instructions

Priority order (higher overrides lower):
1. `CLAUDE.md` in directories of files being reviewed
2. `CLAUDE.md` in repository root
3. `.ai/AI-AGENT-INSTRUCTIONS.md`
4. `.github/copilot-instructions.md`

For each file being reviewed, identify which instruction files apply (same directory or any parent up to repo root).

### Language Detection

- **Node.js/TypeScript**: `package.json`
- **.NET/C#**: `*.csproj`, `*.sln`, `*.slnx`
- **Override**: `--language nodejs|dotnet|react` for ALL files
- **Monorepo**: Per-file detection; group by language

### Framework Detection

Node.js files: check nearest `package.json` for `react`/`react-dom` in dependencies. `--language react` = Node.js + React checks.

### LSP

Check for LSP plugins: `typescript-lsp` (Node.js), `csharp-lsp`/OmniSharp (.NET). If available: read `${CLAUDE_PLUGIN_ROOT}/shared/references/lsp-integration.md`, add `lsp_available` to discovery. If none: skip.

### Test Files

Find test files per detected language. See `${CLAUDE_PLUGIN_ROOT}/languages/nodejs.md` and `${CLAUDE_PLUGIN_ROOT}/languages/dotnet.md` for patterns. Merge `additional_test_patterns` from settings.

### Discovery Output

```yaml
ai_instructions: [{path, applies_to}]
detected_languages: {language: [file_list]}
detected_frameworks: {framework: [file_list]}
test_files: {source: [tests]}
language_override: string | null
lsp_available: [language_types] | null
```

Lazy loading: only load `languages/*.md` if files detected for that language. Unknown language: warn, skip language checks.
