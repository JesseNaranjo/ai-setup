## Test File Patterns

In addition to Node.js patterns: `*.test.tsx`, files using `@testing-library/react`, files with `render()` from testing-library.

## Category-Specific Checks

These checks are IN ADDITION to Node.js checks. See `${CLAUDE_PLUGIN_ROOT}/languages/nodejs.md` for base Node.js/TypeScript checks.

### Security {#security}

- XSS via dangerouslySetInnerHTML, href javascript: injection
- Insecure iframe embedding — missing sandbox on user-controlled iframes

### Performance {#performance}

- Large lists without virtualization (1000+ items without windowing)
- Object creation in useEffect dependency array without useMemo

### Architecture {#architecture}

- Hooks doing too much — single hooks 100+ lines handling multiple concerns
- Prop drilling (>3 levels) — signals need for context or composition

### Test Coverage {#tests}

- Missing async operation tests — useEffect fetching without wait/findBy

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
