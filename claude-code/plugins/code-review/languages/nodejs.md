# Node.js / TypeScript Language Configuration

Language-specific checks and patterns for Node.js and TypeScript projects.

## Detection

Detect Node.js projects by checking for `package.json` in the repository root or parent directories.

## Runtime Detection

Detect runtime environment for runtime-specific checks:

| Runtime | Detection | Special Checks |
|---------|-----------|----------------|
| Bun | `bun.lockb` or `bunfig.toml` | Bun namespace, SQLite |
| Browser | No runtime markers + DOM usage | Window, document patterns |
| Deno | `deno.json` or `deno.jsonc` | Deno namespace, permissions |
| Node.js | `package.json` engines.node | Buffer, fs, process patterns |

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

## Language Server Integration (Optional)

> **LSP Integration (agent-only):** See `${CLAUDE_PLUGIN_ROOT}/shared/references/lsp-integration.md` for TypeScript LSP integration details. Not loaded during orchestration.

## Category-Specific Checks

### Bugs {#bugs}

- Unhandled promise rejections — promises without `.catch()` or in async functions without try/catch
- `undefined`/`null` issues — accessing properties on potentially null/undefined values
- Incorrect `this` binding — arrow functions vs regular functions in callbacks, class methods
- Async/await pitfalls — missing `await`, returning instead of awaiting, parallel vs sequential execution
- Type coercion bugs — loose equality (`==`), implicit type conversions
- Event loop blocking
- Promise.all partial failure — use Promise.allSettled where appropriate
- JSON.parse without try-catch
- Array method on possibly empty — calling .reduce() on potentially empty arrays without initial value
- Timezone-naive Date operations — using new Date() without timezone consideration in server code

### Security {#security}

- Prototype pollution — Object.assign with user input, recursive merge without safeguards
- ReDoS — regular expressions vulnerable to catastrophic backtracking
- Dynamic code execution — code evaluation functions, dynamic require with user input
- Insecure dependencies
- JWT validation — missing signature verification, weak algorithms, improper token storage
- XSS via template literals — unescaped user input in template strings used in HTML
- Command injection
- Path traversal
- Server-side request forgery (SSRF) — user input in fetch/axios URLs without allowlist validation
- Mass assignment — Object.assign/spread with user input to model objects
- Sensitive data in error messages
- Express helmet missing
- Rate limiting absent

### Performance {#performance}

- Event loop blocking — CPU-intensive operations without worker threads, sync I/O in async context
- Memory leaks — unclosed event listeners, closures capturing large objects, unbounded caches
- Inefficient array methods — `forEach` in hot paths where `for` loop is faster, repeated `find()`/`filter()`
- Missing stream usage
- N+1 queries

### Architecture {#architecture}

- Barrel file abuse — re-exports causing bundle size issues
- Circular imports
- CommonJS vs ESM issues — mixing `require()` and `import`, incorrect file extensions
- God modules
- Improper TypeScript typing — `any` abuse, incorrect type assertions, missing generics
- Missing noImplicitAny
- Missing noUncheckedIndexedAccess
- Missing strictNullChecks
- Strict mode not enabled — tsconfig.json without `"strict": true`

### Error Handling {#errors}

- Unhandled promise rejections
- Missing `.catch()`
- Swallowed errors — empty catch blocks, catch blocks that only log
- Improper error propagation
- Missing finally cleanup

### Test Coverage {#tests}

- Missing unit tests
- Missing integration tests
- Missing edge case tests
- Async test issues
- Test isolation — tests sharing state, order-dependent tests

### Technical Debt {#debt}

- Deprecated dependencies — packages with npm deprecation warnings, major version 2+ behind
- Callback patterns
- CommonJS in ESM — `require()` usage in ESM-configured projects (`"type": "module"`)
- Legacy bundler config — Webpack 4 config, Gulp/Grunt in modern projects
- Outdated TypeScript — TS <4.0 patterns, pre-strict mode code, excessive `any` usage
- TODO/FIXME debt
- Commented code — large blocks of commented-out code (10+ lines)
- Event emitter abuse
- Monolithic modules — single files with 1000+ lines or 50+ exports

## Framework-Specific Checks

Apply these checks when the corresponding framework is detected by context-discovery.

### Express Checks

- Missing error middleware — no app.use((err, req, res, next)) handler
- Route parameter injection — req.params used in SQL/shell without validation
- Trust proxy misconfiguration
- Body parser limits — express.json() without size limits

### NestJS Checks

- Missing validation pipe
- Injectable scope issues — singleton services holding request-scoped data
- Circular module imports — ModuleRef required due to circular DI
- Missing guards on controllers
