## Test File Patterns

- `*.test.ts`
- `*.spec.ts`
- `*.test.js`
- `*.spec.js`
- `*-test.js`
- `*-spec.js`
- `__tests__/` directories
- `tests/` directories

## Modern JavaScript/TypeScript Patterns

### ES2022+ Features to Prefer

| Modern Pattern | Legacy Pattern | Category |
|----------------|----------------|----------|
| `?.` optional chaining | `x && x.y && x.y.z` | Architecture |
| `??` nullish coalescing | `x \|\| defaultValue` (falsy trap) | Bugs |
| `Array.at(-1)` | `arr[arr.length - 1]` | Architecture |
| `Object.hasOwn()` | `hasOwnProperty.call()` | Security |
| Private class fields `#` | WeakMap or naming convention | Architecture |
| Static class blocks | Constructor initialization | Architecture |
| Top-level await | Wrapper async IIFE | Architecture |

### ESM vs CommonJS

- __dirname/__filename in ESM — using CommonJS globals in ESM context
- Default export interop — CJS default imports from ESM
- Dynamic import() for CommonJS — using require() where import() needed
- .mjs/.cjs extension issues — wrong extension for module type
- Package.json "type" mismatch — files using wrong module syntax for package type

## Category-Specific Checks

### Bugs {#bugs}

- Unhandled promise rejections — missing `.catch()` or try/catch around await
- `undefined`/`null` property access
- Incorrect `this` binding — arrow vs regular functions in callbacks/class methods
- Async/await pitfalls — missing `await`, parallel vs sequential execution
- Type coercion bugs — loose equality (`==`), implicit conversions
- Event loop blocking
- Promise.all partial failure — use Promise.allSettled where appropriate
- JSON.parse without try-catch
- Array .reduce() on possibly empty array without initial value
- Timezone-naive Date operations in server code

### Security {#security}

- Prototype pollution — Object.assign/recursive merge with user input
- ReDoS — regexes vulnerable to catastrophic backtracking
- Dynamic code execution (eval, dynamic require with user input)
- JWT validation — missing signature verification, weak algorithms, improper storage
- XSS via template literals — unescaped user input in HTML template strings
- Command injection
- Path traversal
- SSRF — user input in fetch/axios URLs without allowlist
- Mass assignment — spread/Object.assign with user input to model objects
- Sensitive data in error messages
- Express helmet missing
- Rate limiting absent

### Performance {#performance}

- Event loop blocking — CPU-intensive ops without worker threads, sync I/O in async context
- Memory leaks — unclosed listeners, closures capturing large objects, unbounded caches
- Inefficient array methods — `forEach` in hot paths, repeated `find()`/`filter()`
- Missing stream usage
- N+1 queries

### Architecture {#architecture}

- Barrel file abuse — re-exports causing bundle size issues
- Circular imports
- CommonJS/ESM mixing — `require()` and `import` in same project, wrong extensions
- God modules
- `any` abuse, incorrect type assertions, missing generics
- Missing noImplicitAny
- Missing noUncheckedIndexedAccess
- Missing strictNullChecks
- Strict mode not enabled — tsconfig.json without `"strict": true`

### Error Handling {#errors}

- Unhandled promise rejections
- Missing `.catch()`
- Swallowed errors — empty catch blocks, catch-only-log
- Improper error propagation
- Missing finally cleanup

### Test Coverage {#tests}

- Missing unit/integration/edge case tests
- Async test issues
- Test isolation — shared state, order-dependent tests

### Technical Debt {#debt}

- Deprecated dependencies — npm deprecation warnings, major version 2+ behind
- Callback patterns
- CommonJS in ESM — `require()` in `"type": "module"` projects
- Legacy bundler config — Webpack 4, Gulp/Grunt in modern projects
- Outdated TypeScript — TS <4.0 patterns, pre-strict mode, excessive `any`
- TODO/FIXME debt
- Commented code (10+ lines)
- Event emitter abuse
- Monolithic modules — 1000+ lines or 50+ exports

## Framework-Specific Checks

Apply these checks when the corresponding framework is detected by context-discovery.

### Express Checks

- Missing error middleware — no app.use((err, req, res, next)) handler
- Route parameter injection — req.params in SQL/shell without validation
- Trust proxy misconfiguration
- Body parser limits — express.json() without size limits

### NestJS Checks

- Missing validation pipe
- Injectable scope issues — singleton services holding request-scoped data
- Circular module imports — ModuleRef required due to circular DI
- Missing guards on controllers
