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

| Check | Description |
|-------|-------------|
| __dirname/__filename in ESM | Using CommonJS globals in ESM context |
| Default export interop | CJS default imports from ESM |
| Dynamic import() for CommonJS | Using require() where import() needed |
| .mjs/.cjs extension issues | Wrong extension for module type |
| Package.json "type" mismatch | Files using wrong module syntax for package type |

## Language Server Integration (Optional)

> **LSP Integration:** See `${CLAUDE_PLUGIN_ROOT}/shared/references/lsp-integration.md` for TypeScript LSP integration details.

## Category-Specific Checks

### Bugs {#bugs}

| Issue Type | Description |
|------------|-------------|
| Unhandled promise rejections | Promises without `.catch()` or in async functions without try/catch |
| `undefined`/`null` issues | Accessing properties on potentially null/undefined values |
| Incorrect `this` binding | Arrow functions vs regular functions in callbacks, class methods |
| Async/await pitfalls | Missing `await`, returning instead of awaiting, parallel vs sequential execution |
| Type coercion bugs | Loose equality (`==`), implicit type conversions causing unexpected behavior |
| Event loop blocking | Synchronous operations that block the event loop |
| Promise.all partial failure | Using Promise.all where one failure should not abort others (use Promise.allSettled) |
| JSON.parse without try-catch | Parsing untrusted JSON without error handling |
| Array method on possibly empty | Calling .reduce() on potentially empty arrays without initial value |
| Timezone-naive Date operations | Using new Date() without timezone consideration in server code |

### Security {#security}

| Issue Type | Description |
|------------|-------------|
| Prototype pollution | Object.assign with user input, recursive merge without safeguards |
| ReDoS | Regular expressions vulnerable to catastrophic backtracking |
| Dynamic code execution | Code evaluation functions, dynamic require with user input |
| Insecure dependencies | Known vulnerable packages, outdated security-critical dependencies |
| JWT validation | Missing signature verification, weak algorithms, improper token storage |
| XSS via template literals | Unescaped user input in template strings used in HTML |
| Command injection | User input passed to shell execution functions |
| Path traversal | User input in file paths without sanitization |
| Server-side request forgery (SSRF) | User input in fetch/axios URLs without allowlist validation |
| Mass assignment | Object.assign/spread with user input to model objects |
| Sensitive data in error messages | Stack traces or internal details exposed to clients |
| Express helmet missing | Express apps without helmet middleware |
| Rate limiting absent | Public APIs without rate limiting middleware |

### Performance {#performance}

| Issue Type | Description |
|------------|-------------|
| Event loop blocking | CPU-intensive operations without worker threads, sync I/O in async context |
| Memory leaks | Unclosed event listeners, closures capturing large objects, unbounded caches |
| Inefficient array methods | `forEach` in hot paths where `for` loop is faster, repeated `find()`/`filter()` |
| Missing stream usage | Reading large files entirely into memory instead of streaming |
| N+1 queries | Sequential database calls in loops instead of batch queries |

### Architecture {#architecture}

| Issue Type | Description |
|------------|-------------|
| Barrel file abuse | Re-exports causing bundle size issues |
| Circular imports | Modules that import each other causing initialization issues |
| CommonJS vs ESM issues | Mixing `require()` and `import`, incorrect file extensions |
| God modules | Single files with too many responsibilities |
| Improper TypeScript typing | `any` abuse, incorrect type assertions, missing generics |
| Missing noImplicitAny | Implicit any hiding type errors |
| Missing noUncheckedIndexedAccess | Array/object access possibly undefined |
| Missing strictNullChecks | Nullable errors not caught at compile time |
| Strict mode not enabled | tsconfig.json without `"strict": true` |

### Error Handling {#errors}

| Issue Type | Description |
|------------|-------------|
| Unhandled promise rejections | Promises without error handling in async code |
| Missing `.catch()` | Promise chains without terminal error handler |
| Swallowed errors | Empty catch blocks, catch blocks that only log |
| Improper error propagation | Catching and not re-throwing when appropriate |
| Missing finally cleanup | Resources not cleaned up on error |

### Test Coverage {#tests}

| Issue Type | Description |
|------------|-------------|
| Missing unit tests | New functions/classes without corresponding test files |
| Missing integration tests | API endpoints without request/response tests |
| Missing edge case tests | Functions without tests for null, empty, boundary values |
| Async test issues | Tests not properly awaiting async operations |
| Test isolation | Tests sharing state, order-dependent tests |

### Technical Debt {#debt}

| Issue Type | Description |
|------------|-------------|
| Deprecated dependencies | Packages with npm deprecation warnings, major version 2+ behind |
| Callback patterns | Callbacks instead of async/await in modern codebases |
| CommonJS in ESM | `require()` usage in ESM-configured projects (`"type": "module"`) |
| Legacy bundler config | Webpack 4 config, Gulp/Grunt in modern projects |
| Outdated TypeScript | TS <4.0 patterns, pre-strict mode code, excessive `any` usage |
| TODO/FIXME debt | Comments without issue tracking references |
| Commented code | Large blocks of commented-out code (10+ lines) |
| Event emitter abuse | Excessive event-driven patterns where direct calls suffice |
| Monolithic modules | Single files with 1000+ lines or 50+ exports |

## Framework-Specific Checks

Apply these checks when the corresponding framework is detected by context-discovery.

### Express Checks

| Issue Type | Description |
|------------|-------------|
| Missing error middleware | No app.use((err, req, res, next)) handler |
| Route parameter injection | req.params used in SQL/shell without validation |
| Trust proxy misconfiguration | Missing or incorrect trust proxy behind reverse proxy |
| Body parser limits | express.json() without size limits |

### NestJS Checks

| Issue Type | Description |
|------------|-------------|
| Missing validation pipe | Controllers without ValidationPipe |
| Injectable scope issues | Singleton services holding request-scoped data |
| Circular module imports | ModuleRef required due to circular DI |
| Missing guards on controllers | Endpoints without AuthGuard |
