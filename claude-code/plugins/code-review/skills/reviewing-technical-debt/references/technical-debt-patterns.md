# Technical Debt Patterns Reference

## Deprecated Dependencies

### Node.js

| Pattern | Detection | Severity |
|---------|-----------|----------|
| npm deprecation | `npm ls` "WARN deprecated" | Major |
| Major version 2+ behind | Compare package.json vs registry | Major |
| Unmaintained | No commits 2+ years, archived | Major |
| Known CVE | `npm audit`, Snyk | Critical |

**Common:** `request` (use node-fetch/axios), `moment` (use date-fns/luxon), `node-sass` (use sass), `tslint` (use eslint)

### .NET

| Pattern | Detection | Severity |
|---------|-----------|----------|
| NuGet deprecation | Package marked obsolete | Major |
| .NET Framework only | No .NET Core/5+ support | Major |
| Known CVE | NuGet advisory database | Critical |

**Common:** `Newtonsoft.Json` in .NET 5+ (consider System.Text.Json), `Microsoft.AspNet.*` (use AspNetCore), `WindowsAzure.Storage` (use Azure.Storage.*)

## Outdated Patterns

### Node.js/TypeScript

| Pattern | Detection Regex |
|---------|----------------|
| Callbacks | `function\s+\w+\s*\([^)]*,\s*(?:callback\|cb\|done)\)` |
| Class components | `class\s+\w+\s+extends\s+(?:React\.)?Component` |
| CommonJS in ESM | `(?:const\|let\|var)\s+\w+\s*=\s*require\s*\(` |
| var keyword | `var x =` |

### .NET/C#

| Pattern | Detection Regex |
|---------|----------------|
| WebClient | `new\s+WebClient\s*\(` |
| Sync-over-async | `\.Result\b\|\.Wait\(\)\|\.GetAwaiter\(\)\.GetResult\(\)` |
| BinaryFormatter | `BinaryFormatter\|SoapFormatter` |
| Old configuration | `ConfigurationManager` |

## Workarounds and Hacks

**Detection regex:** `(?:\/\/|#|\/\*)\s*(?:HACK|WORKAROUND|XXX|TEMP(?:ORARY)?)\s*[:\-]?` and `(?:workaround|hack)\s+(?:for|due to)\s+.*(?:version|v\d|bug|issue)`

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
| Abandoned feature flags | Always true/false | Minor |

**Detection regex:** `^(?:\s*(?:\/\/|#).*\n){10,}` and `return\s+[^;]+;\s*\n\s*(?![\}\)])[^\/\n]+`

## Scalability Concerns

Unbounded array (`items.push()` without limit), in-memory cache (`const cache = {}`/`new Map()`), sync file I/O (`fs.readFileSync`, `File.ReadAllText`), sequential when parallel (`for await` on independent ops).

## Documentation Debt

**Detection regex:** `(?:\/\/|#)\s*TODO(?:\([^)]*\))?:?\s+(?!.*(?:#\d+|ISSUE-|JIRA-|TICKET-))` and `(?:\/\/|#)\s*FIXME:?\s+(?!.*(?:#\d+|ISSUE-))`

| Pattern | Classification |
|---------|---------------|
| TODO without issue ref | Debt |
| TODO with issue ref (#123) | Not debt |
| FIXME without tracking | Debt |
| Old TODO (>1 year) | Major debt |
| Comments referencing old versions | Stale |
| Comments with dates 2+ years old | Stale |
