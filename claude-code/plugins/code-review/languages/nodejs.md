# Node.js / TypeScript Language Configuration

Language-specific checks and patterns for Node.js and TypeScript projects.

## Detection

Detect Node.js projects by checking for `package.json` in the repository root or parent directories.

## Test File Patterns

- `*.test.ts`
- `*.spec.ts`
- `*.test.js`
- `*.spec.js`
- `*-test.js`
- `*-spec.js`
- `__tests__/` directories
- `tests/` directories

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

### Performance {#performance}

| Issue Type | Description |
|------------|-------------|
| Event loop blocking | CPU-intensive operations without worker threads, sync I/O in async context |
| Memory leaks | Unclosed event listeners, closures capturing large objects, unbounded caches |
| Inefficient array methods | `forEach` in hot paths where `for` loop is faster, repeated `find()`/`filter()` |
| Missing stream usage | Reading large files entirely into memory instead of streaming |
| Unnecessary re-renders | React: missing memoization, inline functions/objects in props |
| N+1 queries | Sequential database calls in loops instead of batch queries |

### Architecture {#architecture}

| Issue Type | Description |
|------------|-------------|
| CommonJS vs ESM issues | Mixing `require()` and `import`, incorrect file extensions |
| Circular imports | Modules that import each other causing initialization issues |
| React hooks rules violations | Hooks called conditionally, hooks in loops, missing dependencies in useEffect |
| Improper TypeScript typing | `any` abuse, incorrect type assertions, missing generics |
| God modules | Single files with too many responsibilities |
| Barrel file abuse | Re-exports causing bundle size issues |

### Error Handling {#errors}

| Issue Type | Description |
|------------|-------------|
| Unhandled promise rejections | Promises without error handling in async code |
| Missing `.catch()` | Promise chains without terminal error handler |
| Missing error boundaries | React apps without error boundary components |
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
| Class components | React class components in React 18+ projects |
| CommonJS in ESM | `require()` usage in ESM-configured projects (`"type": "module"`) |
| Legacy bundler config | Webpack 4 config, Gulp/Grunt in modern projects |
| Outdated TypeScript | TS <4.0 patterns, pre-strict mode code, excessive `any` usage |
| TODO/FIXME debt | Comments without issue tracking references |
| Commented code | Large blocks of commented-out code (10+ lines) |
| Event emitter abuse | Excessive event-driven patterns where direct calls suffice |
| Monolithic modules | Single files with 1000+ lines or 50+ exports |
