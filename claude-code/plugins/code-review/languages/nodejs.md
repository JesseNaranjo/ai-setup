## Test File Patterns

- `*.test.ts`, `*.spec.ts`, `*.test.js`, `*.spec.js`, `*-test.js`, `*-spec.js`
- `__tests__/`, `tests/` directories

## Category-Specific Checks

### Bugs {#bugs}

- Timezone-naive Date operations in server code

### Security {#security}

- Prototype pollution — Object.assign/recursive merge with user input
- ReDoS — regexes vulnerable to catastrophic backtracking
- JWT validation — missing signature verification, weak algorithms, improper storage
- SSRF — user input in fetch/axios URLs without allowlist
- Express helmet missing, rate limiting absent

### Architecture {#architecture}

- Barrel file abuse — re-exports causing bundle size issues
- `any` abuse, incorrect type assertions, missing generics
- Missing noImplicitAny, noUncheckedIndexedAccess, strictNullChecks in tsconfig

### Technical Debt {#debt}

- Deprecated dependencies — npm deprecation warnings, major version 2+ behind
- Legacy bundler config — Webpack 4, Gulp/Grunt in modern projects
- Monolithic modules — 1000+ lines or 50+ exports

## Framework-Specific Checks

Apply when corresponding framework detected by context-discovery.

### Express Checks

- Missing error middleware — no app.use((err, req, res, next)) handler
- Route parameter injection — req.params in SQL/shell without validation
- Trust proxy misconfiguration
- Body parser limits — express.json() without size limits

### NestJS Checks

- Injectable scope issues — singleton services holding request-scoped data
- Circular module imports — ModuleRef required due to circular DI
- Missing guards on controllers
