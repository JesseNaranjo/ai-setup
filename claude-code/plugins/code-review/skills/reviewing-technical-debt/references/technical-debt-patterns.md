# Technical Debt Patterns Reference

## Deprecated Dependencies

### Node.js

| Pattern | Detection | Severity |
|---------|-----------|----------|
| npm deprecation | `npm ls` "WARN deprecated" | Major |
| Major version 2+ behind | Compare package.json vs registry | Major |
| Unmaintained | No commits 2+ years, archived | Major |
| Known CVE | `npm audit`, Snyk | Critical |

**Common:** `request` (use node-fetch/axios), `moment` (use date-fns/luxon), `node-sass` (use sass), `tslint` (use eslint), `passport` < v0.7 (session fixation fix), `uuid` v3 (use v4+), `create-react-app` (deprecated 2023; use Vite, Next.js, or Remix), `eslint` v8 `.eslintrc` format (v9 requires `eslint.config.js` flat config), `faker` (compromised; use `@faker-js/faker`), `querystring` built-in (deprecated; use `URLSearchParams`)

### .NET

| Pattern | Detection | Severity |
|---------|-----------|----------|
| NuGet deprecation | Package marked obsolete | Major |
| .NET Framework only | No .NET Core/5+ support | Major |
| Known CVE | NuGet advisory database | Critical |

**Common:** `Newtonsoft.Json` in .NET 5+ (consider System.Text.Json), `Microsoft.AspNet.*` (use AspNetCore), `WindowsAzure.Storage` (use Azure.Storage.*), `HttpWebRequest` (use `HttpClient` via `IHttpClientFactory`)

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

## Runtime & Framework Lifecycle

### Node.js LTS Schedule

| Version | Status | EOL |
|---------|--------|-----|
| 16.x | EOL | Sep 2023 |
| 18.x | EOL | Apr 2025 |
| 20.x | Maintenance LTS | Apr 2026 |
| 22.x | Active LTS | Apr 2027 |

**Detection:** Check `engines.node` in `package.json`, `.nvmrc`, `.node-version`, `Containerfile FROM node:<version>`, `Dockerfile FROM node:<version>`.

### .NET Support Lifecycle

| Version | Status | EOL |
|---------|--------|-----|
| .NET 6 | EOL | Nov 2024 |
| .NET 7 | EOL | May 2024 |
| .NET 8 | LTS | Nov 2026 |
| .NET 9 | STS | May 2026 |

**Detection:** Check `<TargetFramework>` in `*.csproj`. `net6.0` = .NET 6, `net8.0` = .NET 8, etc.

### Key Framework Versions

Known major transitions (alphabetical):

- ASP.NET Core: version should match target .NET version
- Bootstrap: 4→5 (dropped jQuery dependency, new utility API, RTL support). Check `package.json` dependencies
- EF Core: version should match target .NET version. Check `*.csproj` PackageReference
- Express: 4→5 (path matching, async error handling). Check `package.json` dependencies
- Fastify: 3→4 (TypeScript-first, hooks API overhaul, serialization changes), 4→5 (Node.js 20+ required). Check `package.json` dependencies
- Next.js: 13→14 (App Router stable), 14→15 (React 19 support). Check `package.json` dependencies
- React: 17→18 (concurrent rendering), 18→19 (compiler, forwardRef removal). Check `package.json` dependencies
- TypeScript: 4.x→5.x (module resolution changes). Check `package.json` devDependencies
- Vite: 4→5 (Node.js 18+ required, environment API), 5→6 (Node.js 20+ required). Check `package.json` devDependencies
