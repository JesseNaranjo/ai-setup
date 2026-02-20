## Test File Patterns

`*.test.ts`, `*.spec.ts`, `*.test.js`, `*.spec.js`, `*-test.js`, `*-spec.js`, `__tests__/`, `tests/`

## Category-Specific Checks

### Bugs {#bugs}

- Timezone-naive Date operations in server code

### Security {#security}

- Prototype pollution — Object.assign/recursive merge with user input
- Missing helmet, missing rate limiting

### Architecture {#architecture}

- Barrel file abuse — re-exports causing bundle bloat
- `any` abuse, incorrect type assertions, missing generics
- Missing noImplicitAny, noUncheckedIndexedAccess, strictNullChecks in tsconfig

### Technical Debt {#debt}

- Deprecated deps — npm deprecation warnings, major version 2+ behind
- Legacy bundler — Webpack 4, Gulp/Grunt in modern projects
- Monolithic modules — 1000+ lines or 50+ exports

## Framework-Specific Checks

Apply when framework detected by context-discovery.

### Express

- Missing error middleware — no app.use((err, req, res, next)) handler
- Route param injection — req.params in SQL/shell without validation
- Trust proxy misconfiguration
- Body parser limits — express.json() without size limits

### NestJS

- Injectable scope — singleton services holding request-scoped data
- Circular module imports — ModuleRef required due to circular DI
- Missing guards on controllers
