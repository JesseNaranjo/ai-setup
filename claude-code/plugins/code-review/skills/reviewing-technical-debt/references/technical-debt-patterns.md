# Technical Debt Patterns Reference

## Deprecated Dependencies

### Node.js

| Pattern | Detection | Severity |
|---------|-----------|----------|
| npm deprecation warning | `npm ls` with "WARN deprecated" | Major |
| Major version 2+ behind | Compare package.json with registry | Major |
| Unmaintained package | No commits in 2+ years, archived | Major |
| Known CVE | `npm audit`, Snyk | Critical |

**Common deprecated:** `request` (use node-fetch/axios), `moment` (use date-fns/luxon), `node-sass` (use sass), `tslint` (use eslint)

### .NET

| Pattern | Detection | Severity |
|---------|-----------|----------|
| NuGet deprecation | Package marked obsolete | Major |
| .NET Framework only | No .NET Core/5+ support | Major |
| Known CVE | NuGet advisory database | Critical |

**Common deprecated:** `Newtonsoft.Json` in .NET 5+ (consider System.Text.Json), `Microsoft.AspNet.*` (use AspNetCore), `WindowsAzure.Storage` (use Azure.Storage.*)

## Outdated Patterns

### Node.js/TypeScript

| Pattern | Detection Regex | Modern Alternative |
|---------|----------------|-------------------|
| Callbacks | `function\s+\w+\s*\([^)]*,\s*(?:callback\|cb\|done)\)` | async/await |
| Class components | `class\s+\w+\s+extends\s+(?:React\.)?Component` | Functional + hooks |
| CommonJS in ESM | `(?:const\|let\|var)\s+\w+\s*=\s*require\s*\(` | ES imports |
| var keyword | `var x =` | const/let |

### .NET/C#

| Pattern | Detection Regex | Modern Alternative |
|---------|----------------|-------------------|
| WebClient | `new\s+WebClient\s*\(` | HttpClient |
| Sync-over-async | `\.Result\b\|\.Wait\(\)\|\.GetAwaiter\(\)\.GetResult\(\)` | await |
| BinaryFormatter | `BinaryFormatter\|SoapFormatter` | System.Text.Json |
| Old configuration | `ConfigurationManager` | IConfiguration |

## Workarounds and Hacks

**Detection regex:**
```
(?:\/\/|#|\/\*)\s*(?:HACK|WORKAROUND|XXX|TEMP(?:ORARY)?)\s*[:\-]?
(?:workaround|hack)\s+(?:for|due to)\s+.*(?:version|v\d|bug|issue)
```

| Marker | Severity |
|--------|----------|
| HACK, WORKAROUND | Major |
| XXX, TEMP/TEMPORARY | Minor |

## Dead Code

| Type | Detection | Severity |
|------|-----------|----------|
| Unused exports | Export not imported anywhere | Minor |
| Commented code | 10+ consecutive comment lines with code syntax | Minor |
| Unreachable code | Code after unconditional return/throw | Minor |
| Abandoned feature flags | Flags always true/false | Minor |

**Detection regex:**
```
^(?:\s*(?:\/\/|#).*\n){10,}
return\s+[^;]+;\s*\n\s*(?![\}\)])[^\/\n]+
```

## Scalability Concerns

| Pattern | Detection | Issue |
|---------|-----------|-------|
| Unbounded array | `items.push()` without limit | Memory growth |
| In-memory cache | `const cache = {}` or `new Map()` | No persistence/sharing |
| Sync file I/O | `fs.readFileSync`, `File.ReadAllText` | Blocks thread |
| Sequential when parallel | `for await` on independent ops | Performance |

## Documentation Debt

**Detection regex:**
```
(?:\/\/|#)\s*TODO(?:\([^)]*\))?:?\s+(?!.*(?:#\d+|ISSUE-|JIRA-|TICKET-))
(?:\/\/|#)\s*FIXME:?\s+(?!.*(?:#\d+|ISSUE-))
```

| Pattern | Classification |
|---------|---------------|
| TODO without issue ref | Debt |
| TODO with issue ref (#123) | Not debt |
| FIXME without tracking | Debt |
| Old TODO (year > 1 year) | Major debt |
| Comments referencing old versions | Stale |
| Comments with dates 2+ years old | Stale |
