# Technical Debt Patterns Reference

Detailed patterns for identifying technical debt across Node.js/TypeScript and .NET/C# projects.

## Deprecated Dependencies

### Node.js Patterns

| Pattern | Detection | Severity |
|---------|-----------|----------|
| npm deprecation warning | `npm ls` output with "WARN deprecated" | Major |
| Major version 2+ behind | Compare package.json versions with npm registry | Major |
| Unmaintained package | No commits in 2+ years, archived repo | Major |
| Known CVE | `npm audit` output, Snyk database | Critical |

**Common deprecated packages:**
- `request` - deprecated 2020, use `node-fetch` or `axios`
- `moment` - maintenance mode, use `date-fns` or `luxon`
- `uuid` v3 or lower - use v4+
- `node-sass` - use `sass` (Dart Sass)
- `tslint` - use `eslint` with TypeScript plugin

### .NET Patterns

| Pattern | Detection | Severity |
|---------|-----------|----------|
| NuGet deprecation | Package marked obsolete in NuGet | Major |
| Discontinued library | No updates in 2+ years | Major |
| .NET Framework only | Package doesn't support .NET Core/5+ | Major |
| Known CVE | NuGet advisory database | Critical |

**Common deprecated packages:**
- `Newtonsoft.Json` in .NET 5+ - consider `System.Text.Json`
- `Microsoft.AspNet.*` - use `Microsoft.AspNetCore.*`
- `System.Web` dependencies - ASP.NET Core alternatives
- `WindowsAzure.Storage` - use `Azure.Storage.*`

---

## Outdated Patterns

### Node.js/TypeScript

| Pattern | Detection | Modern Alternative |
|---------|-----------|-------------------|
| Callbacks | `function(err, result)` signature | async/await |
| Class components | `class X extends React.Component` | Functional components + hooks |
| CommonJS in ESM | `require()` in `"type": "module"` project | ES imports |
| var keyword | `var x =` | `const` / `let` |
| Webpack 4 config | `webpack.config.js` with v4 syntax | Webpack 5 or Vite |
| Gulp/Grunt | `gulpfile.js`, `Gruntfile.js` | npm scripts or modern bundlers |
| Pre-strict TypeScript | Missing `strict: true` in tsconfig | Enable strict mode |

**Detection regex:**
```regex
# Callback pattern
function\s+\w+\s*\([^)]*,\s*(?:callback|cb|done|next)\s*\)

# Class component
class\s+\w+\s+extends\s+(?:React\.)?Component

# CommonJS require
(?:const|let|var)\s+\w+\s*=\s*require\s*\(
```

### .NET/C#

| Pattern | Detection | Modern Alternative |
|---------|-----------|-------------------|
| WebClient | `new WebClient()` | `HttpClient` |
| Sync-over-async | `.Result`, `.Wait()`, `.GetAwaiter().GetResult()` | `await` |
| Old configuration | `ConfigurationManager` | `IConfiguration` |
| BinaryFormatter | `BinaryFormatter` serialization | `System.Text.Json` |
| Pre-nullable | Missing `#nullable enable` | Nullable reference types |
| Task.Run for async | `Task.Run(() => SyncMethod())` | Native async |

**Detection regex:**
```regex
# Sync-over-async
\.Result\b|\.Wait\(\)|\.GetAwaiter\(\)\.GetResult\(\)

# Old WebClient
new\s+WebClient\s*\(

# BinaryFormatter
BinaryFormatter|SoapFormatter
```

---

## Workarounds and Hacks

### Detection Patterns

| Marker | Regex | Severity |
|--------|-------|----------|
| HACK comment | `HACK:|HACK\b` | Major |
| WORKAROUND comment | `WORKAROUND:|WORKAROUND\b` | Major |
| XXX marker | `XXX:|XXX\b` | Minor |
| Temporary fix | `TEMP:|TEMPORARY:` | Minor |
| Version workaround | `workaround for .* version` (case-insensitive) | Minor |

**Detection regex:**
```regex
# Explicit markers
(?:\/\/|#|\/\*)\s*(?:HACK|WORKAROUND|XXX|TEMP(?:ORARY)?)\s*[:\-]?

# Version-specific workaround
(?:workaround|hack)\s+(?:for|due to)\s+.*(?:version|v\d|bug|issue)
```

### Common Workaround Patterns

- Monkey-patching prototype or built-in objects
- Runtime type checks that should be compile-time
- Duplicate code to avoid circular imports
- Environment-specific branches for library bugs

---

## Dead Code

### Detection Patterns

| Type | Detection Method | Severity |
|------|------------------|----------|
| Unused exports | Export not imported anywhere in project | Minor |
| Commented code | 10+ consecutive comment lines with code syntax | Minor |
| Unreachable code | Code after unconditional return/throw | Minor |
| Abandoned feature flags | Feature flags always true/false | Minor |

**Detection regex:**
```regex
# Commented out code block (10+ lines)
^(?:\s*(?:\/\/|#).*\n){10,}

# Unreachable after return
return\s+[^;]+;\s*\n\s*(?![\}\)])[^\/\n]+

# Always-true feature flag
(?:if|when)\s*\(\s*(?:true|1)\s*\)
```

### Cross-File Detection

For unused exports:
1. Find all `export` statements
2. Grep for import of that symbol across codebase
3. Flag if zero imports found (excluding the file itself)

---

## Scalability Concerns

| Pattern | Detection | Issue |
|---------|-----------|-------|
| Unbounded array | `items.push()` without limit check | Memory growth |
| Hardcoded limit | `const MAX = 100` | Won't scale |
| In-memory cache | `const cache = {}` or `new Map()` | No persistence/sharing |
| Sync file I/O | `fs.readFileSync`, `File.ReadAllText` | Blocks thread |
| Sequential when parallel | `for await` on independent operations | Performance |

---

## Documentation Debt

### TODO/FIXME Patterns

| Pattern | Regex | Classification |
|---------|-------|----------------|
| Untracked TODO | `TODO(?!.*#\d+)(?!.*ISSUE-)(?!.*JIRA-)` | Debt |
| Tracked TODO | `TODO.*(?:#\d+\|ISSUE-\|JIRA-)` | Not debt |
| Untracked FIXME | `FIXME(?!.*#\d+)` | Debt |
| Old TODO | `TODO.*\d{4}` (year reference > 1 year old) | Major debt |

**Detection regex:**
```regex
# TODO without issue reference
(?:\/\/|#)\s*TODO(?:\([^)]*\))?:?\s+(?!.*(?:#\d+|ISSUE-|JIRA-|TICKET-))

# FIXME without tracking
(?:\/\/|#)\s*FIXME:?\s+(?!.*(?:#\d+|ISSUE-))
```

### Stale Comments

- Comments referencing old versions (`v1`, `v2` when on v5)
- Comments with dates more than 2 years old
- Comments describing behavior that no longer matches code
