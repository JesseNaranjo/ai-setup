## Test File Patterns

In addition to Node.js patterns:
- `*.test.tsx` for React component tests
- Files using `@testing-library/react`
- Files with `render()` from testing-library

## Category-Specific Checks

These checks are IN ADDITION to Node.js checks. See `${CLAUDE_PLUGIN_ROOT}/languages/nodejs.md` for base Node.js/TypeScript checks.

### Bugs {#bugs}

- Stale closures in hooks — state values captured at definition time, not updated in callbacks/intervals
- Incorrect dependency arrays — infinite loops from object/array deps without memoization
- State updates on unmounted components
- Controlled/uncontrolled input mixing
- Key prop issues — missing keys, index as key for dynamic lists
- Ref mutations during render — mutating refs in render instead of useEffect
- useEffect async function — async defined inside useEffect (should be separate)
- Derived state anti-pattern — useState for values computable from props
- Context value object recreation — new object in Provider value each render

### Security {#security}

- XSS via dangerouslySetInnerHTML, href javascript: injection
- Sensitive data in client state
- Insecure iframe embedding — missing sandbox on user-controlled iframes
- localStorage/sessionStorage sensitive data

### Performance {#performance}

- Context provider at top level — changes causing entire app re-render
- Inline objects/arrays in props — new references every render causing child re-renders
- Large lists without virtualization (1000+ items without windowing)
- Missing code splitting
- Missing React.memo / useCallback / useMemo where beneficial
- Heavy computation in render
- Object creation in useEffect dependency array without useMemo
- Large bundle imports — full library imports instead of tree-shakeable

### Architecture {#architecture}

- Circular component dependencies
- Component does too much — mixing presentation, logic, and data fetching
- Hooks doing too much — single hooks 100+ lines handling multiple concerns
- Missing custom hooks extraction
- Prop drilling (>3 levels)
- React hooks rules violations — conditional hooks, loops, missing useEffect deps

### Error Handling {#errors}

- Error boundaries too high — single boundary catching all errors
- Missing error boundaries / fallbacks, loading/error states
- Suspense without error boundaries
- Uncaught async errors in useEffect

### Test Coverage {#tests}

- Missing component tests
- Testing implementation details — checking internal state instead of behavior
- Missing user interaction tests, accessibility tests
- Snapshot tests without behavior tests
- Missing async operation tests — useEffect fetching without wait/findBy

### Technical Debt {#debt}

- Class components — use functional components in React 18+
- Deprecated React APIs — ReactDOM.render instead of createRoot (React 18+)
- Excessive HOCs, findDOMNode usage
- Legacy context API (childContextTypes)
- Legacy lifecycle methods — componentWillMount, componentWillReceiveProps, componentWillUpdate
- PropTypes in TypeScript projects, string refs

## State Management Checks

Apply when state management libraries detected by context-discovery.

### Redux/RTK Patterns

- Direct state mutation
- Selector recomputation — selectors without createSelector memoization
- Non-serializable state — functions/class instances in Redux state
- Missing loading/error states in async thunks

### React Query Patterns

- Missing error boundaries for queries
- Stale time too short — refetching on every mount unnecessarily
- Missing query invalidation

## Next.js Specific Checks

Apply when Next.js is detected by context-discovery (presence of `next` in deps).

### App Router (Server Components)

- Unnecessary 'use client' on component with no hooks/events
- useState in server component
- Missing Suspense for async server components (no loading.tsx)
- Large data passed from server to client components

### API Routes

- Missing authentication, rate limiting
- Improper error responses
