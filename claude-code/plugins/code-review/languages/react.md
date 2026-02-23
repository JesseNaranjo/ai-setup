## Test File Patterns

Node.js patterns plus: `*.test.tsx`, `@testing-library/react` files, `render()` from testing-library.

## Category-Specific Checks

IN ADDITION to Node.js checks. See `${CLAUDE_PLUGIN_ROOT}/languages/nodejs.md` for base checks.

### Architecture {#architecture}

- Hooks doing too much — 100+ lines, multiple concerns
- Prop drilling (>3 levels) — needs context or composition

### Bugs {#bugs}

- key={index} on lists that reorder/filter (stale state preservation)
- Incorrect dependency array — missing deps causing stale closure
- Missing alt text on `<img>`, missing aria-label on icon-only buttons
- Non-semantic interactive elements: `<div onClick>` without role="button" and keyboard handlers
- Missing form labels: `<input>` without associated `<label>` or aria-label
- Focus management: modal/dialog opens without focus trap, route changes without focus reset
- Invalid ARIA: misspelled `aria-*` attributes, wrong `role` values, redundant roles on semantic elements (`role="button"` on `<button>`)
- Hardcoded color values without CSS custom properties or theme tokens — breaks dark mode and theming
- Missing skip navigation link in layout components

### Error Handling {#errors}

- Missing Error Boundary wrapping for components with async data fetching or third-party integrations
- useEffect cleanup missing — subscriptions, timers, event listeners, AbortController for fetch
- Async errors in event handlers: `onClick={async () => {}}` without try/catch (unhandled rejection, no user feedback)
- Missing Suspense fallback for React.lazy components — white screen on chunk load failure
- Error state not reset on navigation — stale error UI persists across route changes

### Performance {#performance}

- Large lists without virtualization (1000+ items)
- Object creation in useEffect deps without useMemo
- React.memo wrapping components that receive new object/function props every render (memo useless without stable props)

### Security {#security}

- dangerouslySetInnerHTML with unsanitized user input — bypasses React XSS protection; require DOMPurify or equivalent
- href javascript: injection
- Insecure iframe — missing sandbox on user-controlled iframes

### Technical Debt {#debt}

- Class components with no state or lifecycle justifying class form — convert to function
- Deprecated lifecycle methods: componentWillMount, componentWillReceiveProps, componentWillUpdate (use alternatives since React 16.3)
- Legacy context API (contextTypes/childContextTypes) — migrate to createContext/useContext
- String refs (`ref="myRef"`) — migrate to useRef/createRef
- defaultProps on function components (deprecated React 18.3+) — use default parameter values
- PropTypes runtime validation in TypeScript projects — remove in favor of static types

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
- Importing client-only libraries (useState, useEffect, browser APIs) in files without 'use client' — runtime error not caught at build
- Non-serializable props (functions, class instances, Dates) from Server to Client components — silent serialization failure
- fetch() in Server Components: cache default changed between Next 14 (force-cache) and 15 (no-store) — verify explicit revalidate option
- Server Actions ('use server') returning sensitive data — return values serialized to client, visible in network tab
- Layout vs page data fetching: layouts fetch data shared across child routes; page-specific data belongs in pages, not layouts
- Missing `"use server"` directive on exported server-side functions (causes client bundle inclusion)

### Performance

- Route Handler missing `revalidate` export or cache-control headers — defaults to dynamic rendering on every request

### Middleware

- Auth bypass via route ordering: middleware.ts matcher must cover all protected routes, not just a subset
- Overly permissive matchers: `/((?!api|_next|static).*)` may miss dynamically registered routes
- Missing CSP headers in middleware response

### API Routes

- Raw exceptions instead of NextResponse.json() with status codes
- Server action without input validation (Zod/yup)
- Route Handler missing CORS headers when accessed cross-origin
- Route Handler missing auth checks (unlike pages, Route Handlers have no layout-level auth inheritance)
- GET Route Handler with mutation side effects: GET responses may be cached by CDN/browser, causing stale mutations
