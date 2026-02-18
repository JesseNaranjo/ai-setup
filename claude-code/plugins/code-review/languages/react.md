## Test File Patterns

Node.js patterns plus: `*.test.tsx`, `@testing-library/react` files, `render()` from testing-library.

## Category-Specific Checks

IN ADDITION to Node.js checks. See `${CLAUDE_PLUGIN_ROOT}/languages/nodejs.md` for base checks.

### Security {#security}

- XSS via dangerouslySetInnerHTML, href javascript: injection
- Insecure iframe — missing sandbox on user-controlled iframes

### Performance {#performance}

- Large lists without virtualization (1000+ items)
- Object creation in useEffect deps without useMemo

### Architecture {#architecture}

- Hooks doing too much — 100+ lines, multiple concerns
- Prop drilling (>3 levels) — needs context or composition

### Test Coverage {#tests}

- Missing async tests — useEffect fetching without wait/findBy

## State Management Checks

Apply when state management library detected.

### Redux/RTK

- Selector recomputation — missing createSelector memoization
- Non-serializable state — functions/class instances in Redux state

### React Query

- Stale time too short — unnecessary refetch on every mount
- Missing query invalidation after mutations

## Next.js Specific Checks

Apply when `next` in dependencies.

### App Router (Server Components)

- Unnecessary 'use client' on component with no hooks/events
- Missing Suspense for async server components (no loading.tsx)

### API Routes

- Raw exceptions instead of NextResponse.json() with status codes
