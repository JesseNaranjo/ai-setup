## Test File Patterns

Node.js patterns plus: `*.test.tsx`, `@testing-library/react` files, `render()` from testing-library.

## Category-Specific Checks

IN ADDITION to Node.js checks. See `${CLAUDE_PLUGIN_ROOT}/languages/nodejs.md` for base checks.

### Architecture {#architecture}

- Hooks doing too much — 100+ lines, multiple concerns
- Prop drilling (>3 levels) — needs context or composition
- Barrel file re-exports blocking tree shaking — named re-exports force bundler to include entire module graph
- Missing dynamic imports for heavy components — `React.lazy()` or `next/dynamic` for client-only/large components
- [Next.js] Unnecessary 'use client' on component with no hooks/events
- [Next.js] Layout vs page data fetching: layouts fetch data shared across child routes; page-specific data belongs in pages, not layouts

### Bugs {#bugs}

- Incorrect dependency array — missing deps causing stale closure
- Missing alt text on `<img>`, missing aria-label on icon-only buttons
- Non-semantic interactive elements: `<div onClick>` without role="button" and keyboard handlers
- Missing form labels: `<input>` without associated `<label>` or aria-label
- Focus management: modal/dialog opens without focus trap, route changes without focus reset
- Invalid ARIA: misspelled `aria-*` attributes, wrong `role` values, redundant roles on semantic elements (`role="button"` on `<button>`)
- Hardcoded color values without CSS custom properties or theme tokens — breaks dark mode and theming
- Missing skip navigation link in layout components
- [Redux] Non-serializable state — functions/class instances in Redux state
- [React Query] Missing query invalidation after mutations
- [Next.js] Importing client-only libraries (useState, useEffect, browser APIs) in files without 'use client' — runtime error not caught at build
- [Next.js] Non-serializable props (functions, class instances, Dates) from Server to Client components — silent serialization failure
- [Next.js] Next.js 15: `cookies()`, `headers()`, `params`, `searchParams` are now async — synchronous access throws runtime error. Must `await cookies()` etc.
- [Next.js] GET Route Handler with mutation side effects: GET responses may be cached by CDN/browser, causing stale mutations
- [Server Actions] `useActionState`/`useFormStatus` race conditions with concurrent Server Action submissions — UI state desyncs when multiple actions complete out of order
- [Server Actions] Server Action used as ad-hoc API endpoint without rate limiting or input validation — bypasses API gateway protections

### Error Handling {#errors}

- Missing Error Boundary wrapping for components with async data fetching or third-party integrations
- useEffect cleanup missing — subscriptions, timers, event listeners, AbortController for fetch
- Async errors in event handlers: `onClick={async () => {}}` without try/catch (unhandled rejection, no user feedback)
- Missing Suspense fallback for React.lazy components — white screen on chunk load failure
- Error state not reset on navigation — stale error UI persists across route changes
- [Next.js] Missing Suspense for async server components (no loading.tsx)
- [Next.js] Raw exceptions instead of NextResponse.json() with status codes

### Performance {#performance}

- Large lists without virtualization (1000+ items)
- Object creation in useEffect deps without useMemo
- React.memo wrapping components that receive new object/function props every render (memo useless without stable props)
- [Redux] Selector recomputation — missing createSelector memoization
- [React Query] Stale time too short — unnecessary refetch on every mount
- [Next.js] Next.js 15: `fetch` no longer cached by default (was force-cache in 14) — add explicit `{ cache: 'force-cache' }` or `revalidate` if caching intended
- [Next.js] Route Handler missing `revalidate` export or cache-control headers — defaults to dynamic rendering on every request

### Security {#security}

- dangerouslySetInnerHTML with unsanitized user input — bypasses React XSS protection; require DOMPurify or equivalent
- href javascript: injection
- Insecure iframe — missing sandbox on user-controlled iframes
- [Next.js] Server Actions ('use server') returning sensitive data — return values serialized to client, visible in network tab
- [Next.js] Missing `"use server"` directive on exported server-side functions (causes client bundle inclusion)
- [Next.js] Auth bypass via route ordering: middleware.ts matcher must cover all protected routes, not just a subset
- [Next.js] Overly permissive matchers: `/((?!api|_next|static).*)` may miss dynamically registered routes
- [Next.js] Missing CSP headers in middleware response
- [Next.js] Server action without input validation (Zod/yup)
- [Next.js] Route Handler missing CORS headers when accessed cross-origin
- [Next.js] Route Handler missing auth checks (unlike pages, Route Handlers have no layout-level auth inheritance)
- [Server Actions] Server Action accepting plain POST without CSRF token — unlike API routes, Server Actions accept plain POST requests without built-in CSRF protection; validate origin header or use framework CSRF middleware
- [Server Actions] `redirect()` with user-controlled destination after mutation — open redirect via Server Action response; validate redirect target against allowlist
- [Server Actions] Server Action reading `cookies()`/`headers()` without re-validating authentication — stale session, auth bypass on direct POST

### Technical Debt {#debt}

- Class components with no state or lifecycle justifying class form — convert to function
- Deprecated lifecycle methods: componentWillMount, componentWillReceiveProps, componentWillUpdate (use alternatives since React 16.3)
- Legacy context API (contextTypes/childContextTypes) — migrate to createContext/useContext
- String refs (`ref="myRef"`) — migrate to useRef/createRef
- defaultProps on function components (deprecated React 18.3+) — use default parameter values
- PropTypes runtime validation in TypeScript projects — remove in favor of static types
- React 19: `forwardRef` deprecated — ref is now a regular prop, remove forwardRef wrappers
- React 19: `use()` hook replaces useEffect-for-data-fetching and useContext — flag patterns `use()` directly replaces

### Test Coverage {#tests}

- Missing async tests — useEffect fetching without wait/findBy
