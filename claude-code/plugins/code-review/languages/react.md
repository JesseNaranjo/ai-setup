## Test File Patterns

In addition to Node.js patterns: `*.test.tsx`, files using `@testing-library/react`, files with `render()` from testing-library.

## Category-Specific Checks

These checks are IN ADDITION to Node.js checks. See `${CLAUDE_PLUGIN_ROOT}/languages/nodejs.md` for base Node.js/TypeScript checks.

### Bugs {#bugs}

- Stale closures in hooks — state captured at definition time, not updated in callbacks/intervals
- Incorrect dependency arrays — infinite loops from object/array deps without memoization
- useEffect async function — async defined inside useEffect (should be separate function)
- Derived state anti-pattern — useState for values computable from props
- Key prop issues — index as key for dynamic lists (unstable identity)
- Context value object recreation — new object in Provider value each render

### Security {#security}

- XSS via dangerouslySetInnerHTML, href javascript: injection
- Insecure iframe embedding — missing sandbox on user-controlled iframes

### Performance {#performance}

- Context provider at top level — changes causing entire app re-render
- Inline objects/arrays in props — new references every render causing child re-renders
- Large lists without virtualization (1000+ items without windowing)
- Object creation in useEffect dependency array without useMemo

### Architecture {#architecture}

- Component does too much — mixing presentation, logic, and data fetching
- Hooks doing too much — single hooks 100+ lines handling multiple concerns
- Prop drilling (>3 levels) — signals need for context or composition
- React hooks rules violations — conditional hooks, loops, missing useEffect deps

### Error Handling {#errors}

- Missing error boundaries / fallbacks for async operations
- Suspense without error boundaries

### Test Coverage {#tests}

- Testing implementation details — checking internal state instead of behavior
- Missing async operation tests — useEffect fetching without wait/findBy

### Technical Debt {#debt}

- Class components — use functional components in React 18+
- Deprecated React APIs — ReactDOM.render instead of createRoot (React 18+)

## State Management Checks

Apply when state management library detected.

### Redux/RTK Patterns

- Selector recomputation — selectors without createSelector memoization
- Non-serializable state — functions/class instances in Redux state

### React Query Patterns

- Stale time too short — refetching on every mount unnecessarily
- Missing query invalidation after mutations

## Next.js Specific Checks

Apply when Next.js detected (`next` in dependencies).

### App Router (Server Components)

- Unnecessary 'use client' on component with no hooks/events
- Missing Suspense for async server components (no loading.tsx)

### API Routes

- Improper error responses — raw exceptions instead of NextResponse.json() with status codes
