# React Language Configuration

Language-specific checks and patterns for React projects. React projects also receive all Node.js/TypeScript checks from `${CLAUDE_PLUGIN_ROOT}/languages/nodejs.md`.

## Detection

Detect React projects by checking for:
- `package.json` containing `react` or `react-dom` in `dependencies` or `devDependencies`
- Files with `.jsx` or `.tsx` extensions
- Imports from `react` or `react-dom` packages

## Framework Detection Patterns

| Pattern | Confidence | Description |
|---------|------------|-------------|
| `"react"` in package.json deps | High | Standard React dependency |
| `*.jsx` files present | Medium | JSX file extension |
| `'react-dom'` in dependencies | High | React DOM package |

## Test File Patterns

In addition to Node.js patterns:
- `*.test.tsx` for React component tests
- Files using `@testing-library/react`
- Files with `render()` from testing-library

## Category-Specific Checks

These checks are IN ADDITION to Node.js checks. See `${CLAUDE_PLUGIN_ROOT}/languages/nodejs.md` for base Node.js/TypeScript checks.

### Bugs {#bugs}

| Issue Type | Description |
|------------|-------------|
| Stale closures in hooks | State values captured at definition time, not updated in callbacks/intervals |
| Incorrect dependency arrays | Infinite loops from object/array dependencies without memoization |
| State updates on unmounted components | Async operations updating state after component unmount |
| Controlled/uncontrolled input mixing | Switching between controlled and uncontrolled inputs |
| Key prop issues | Missing keys in lists, using index as key for dynamic lists |
| Ref updates during render | Mutating refs during render instead of in useEffect |
| useEffect async function | Defining async function inside useEffect (should be separate) |
| Derived state anti-pattern | useState for values computable from props |
| Context value object recreation | Passing new object to Context.Provider value on each render |

### Security {#security}

| Issue Type | Description |
|------------|-------------|
| XSS via dangerouslySetInnerHTML | Unescaped user input in dangerouslySetInnerHTML |
| href javascript: injection | User input in href attributes allowing javascript: URLs |
| Sensitive data in client state | Passwords, tokens, PII stored in React state |
| Insecure iframe embedding | Missing sandbox attributes on user-controlled iframes |
| localStorage/sessionStorage sensitive data | Tokens or credentials in browser storage without encryption |

### Performance {#performance}

| Issue Type | Description |
|------------|-------------|
| Context provider at top level | Context changes causing entire app re-render |
| Inline objects/arrays in props | New object/array references on every render causing child re-renders |
| Large component trees without virtualization | Rendering 1000+ list items without windowing |
| Missing code splitting | Large bundles without React.lazy/Suspense |
| Missing React.memo | Components receiving same props re-rendering frequently |
| Missing useCallback for handler props | Handler functions recreated on every render |
| Missing useMemo for expensive computations | Heavy calculations running on every render |
| Unnecessary re-renders | Missing memoization, inline functions/objects in props causing child re-renders |
| Heavy computation in render | Complex calculations not memoized with useMemo |
| Object creation in dependency array | Objects in useEffect deps without useMemo |
| Large bundle imports | Full library imports instead of tree-shakeable |

### Architecture {#architecture}

| Issue Type | Description |
|------------|-------------|
| Circular component dependencies | Components importing each other |
| Component does too much | Components mixing presentation, business logic, and data fetching |
| Hooks doing too much | Single hooks with 100+ lines handling multiple concerns |
| Incorrect component boundaries | Components that are too granular or too large |
| Missing custom hooks extraction | Repeated hook patterns across components |
| Prop drilling (>3 levels) | Props passed through multiple intermediate components |
| React hooks rules violations | Hooks called conditionally, hooks in loops, missing dependencies in useEffect |

### Error Handling {#errors}

| Issue Type | Description |
|------------|-------------|
| Error boundaries too high | Single boundary catching all errors, preventing granular recovery |
| Missing error boundaries | React apps without error boundary components at critical points |
| Missing error boundary fallbacks | Error boundaries without user-friendly fallback UI |
| Missing loading/error states | Data fetching without error state handling |
| Suspense without error boundaries | React Suspense without accompanying error boundaries |
| Uncaught async errors in useEffect | Promise rejections not caught in effects |

### Test Coverage {#tests}

| Issue Type | Description |
|------------|-------------|
| Missing component tests | Components without render/interaction tests |
| Testing implementation details | Tests checking internal state instead of behavior |
| Missing user interaction tests | Click/input handlers without event simulation tests |
| Snapshot tests without behavior tests | Over-reliance on snapshots |
| Missing async operation tests | useEffect data fetching without wait/findBy tests |
| Missing accessibility tests | No a11y testing for interactive components |

### Technical Debt {#debt}

| Issue Type | Description |
|------------|-------------|
| Class components | React class components in React 18+ projects (should use functional components) |
| Deprecated React APIs | Using ReactDOM.render instead of createRoot (React 18+) |
| Excessive HOCs | Wrapper hell from many higher-order components |
| findDOMNode usage | Using deprecated findDOMNode |
| Legacy context API | Using old context API (childContextTypes) |
| Legacy lifecycle methods | componentWillMount, componentWillReceiveProps, componentWillUpdate |
| PropTypes in TypeScript | Redundant PropTypes validation in TypeScript projects |
| String refs | Using string refs instead of createRef/useRef |

## State Management Checks

Apply when state management libraries detected by context-discovery.

### Redux/RTK Patterns

| Issue Type | Description |
|------------|-------------|
| Direct state mutation | Modifying state outside of reducers |
| Selector recomputation | Selectors without createSelector memoization |
| Non-serializable state | Functions or class instances in Redux state |
| Missing loading/error states | Async thunks without pending/rejected handling |

### React Query Patterns

| Issue Type | Description |
|------------|-------------|
| Missing error boundaries for queries | Queries without onError or error boundary |
| Stale time too short | Refetching on every mount unnecessarily |
| Missing query invalidation | Mutations not invalidating related queries |

## Next.js Specific Checks

Apply when Next.js is detected by context-discovery (presence of `next` in deps).

### App Router (Server Components)

| Issue Type | Description |
|------------|-------------|
| Client component unnecessary | 'use client' on component with no hooks/events |
| useState in server component | Client hooks used in server components |
| Missing Suspense for async | Async server components without loading.tsx |
| Large data in server components | Passing large objects from server to client |

### API Routes

| Issue Type | Description |
|------------|-------------|
| Missing authentication | API routes without auth middleware |
| Missing rate limiting | Public APIs without rate limits |
| Improper error responses | Exposing internal errors to clients |
