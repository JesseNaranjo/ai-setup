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

- Stale closures in hooks — state values captured at definition time, not updated in callbacks/intervals
- Incorrect dependency arrays — infinite loops from object/array dependencies without memoization
- State updates on unmounted components
- Controlled/uncontrolled input mixing
- Key prop issues — missing keys in lists, using index as key for dynamic lists
- Ref updates during render — mutating refs during render instead of in useEffect
- useEffect async function — defining async function inside useEffect (should be separate)
- Derived state anti-pattern — useState for values computable from props
- Context value object recreation — passing new object to Context.Provider value on each render

### Security {#security}

- XSS via dangerouslySetInnerHTML
- href javascript: injection
- Sensitive data in client state
- Insecure iframe embedding — missing sandbox attributes on user-controlled iframes
- localStorage/sessionStorage sensitive data

### Performance {#performance}

- Context provider at top level — context changes causing entire app re-render
- Inline objects/arrays in props — new references on every render causing child re-renders
- Large component trees without virtualization — rendering 1000+ list items without windowing
- Missing code splitting
- Missing React.memo
- Missing useCallback for handler props
- Missing useMemo for expensive computations
- Unnecessary re-renders — missing memoization, inline functions/objects in props
- Heavy computation in render
- Object creation in dependency array — objects in useEffect deps without useMemo
- Large bundle imports — full library imports instead of tree-shakeable

### Architecture {#architecture}

- Circular component dependencies
- Component does too much — mixing presentation, business logic, and data fetching
- Hooks doing too much — single hooks with 100+ lines handling multiple concerns
- Incorrect component boundaries
- Missing custom hooks extraction
- Prop drilling (>3 levels)
- React hooks rules violations — hooks called conditionally, in loops, missing dependencies in useEffect

### Error Handling {#errors}

- Error boundaries too high — single boundary catching all errors, preventing granular recovery
- Missing error boundaries
- Missing error boundary fallbacks
- Missing loading/error states
- Suspense without error boundaries
- Uncaught async errors in useEffect

### Test Coverage {#tests}

- Missing component tests
- Testing implementation details — tests checking internal state instead of behavior
- Missing user interaction tests
- Snapshot tests without behavior tests — over-reliance on snapshots
- Missing async operation tests — useEffect data fetching without wait/findBy tests
- Missing accessibility tests

### Technical Debt {#debt}

- Class components — should use functional components in React 18+
- Deprecated React APIs — using ReactDOM.render instead of createRoot (React 18+)
- Excessive HOCs
- findDOMNode usage
- Legacy context API — using old context API (childContextTypes)
- Legacy lifecycle methods — componentWillMount, componentWillReceiveProps, componentWillUpdate
- PropTypes in TypeScript
- String refs

## State Management Checks

Apply when state management libraries detected by context-discovery.

### Redux/RTK Patterns

- Direct state mutation
- Selector recomputation — selectors without createSelector memoization
- Non-serializable state — functions or class instances in Redux state
- Missing loading/error states — async thunks without pending/rejected handling

### React Query Patterns

- Missing error boundaries for queries
- Stale time too short — refetching on every mount unnecessarily
- Missing query invalidation

## Next.js Specific Checks

Apply when Next.js is detected by context-discovery (presence of `next` in deps).

### App Router (Server Components)

- Client component unnecessary — 'use client' on component with no hooks/events
- useState in server component — client hooks used in server components
- Missing Suspense for async — async server components without loading.tsx
- Large data in server components — passing large objects from server to client

### API Routes

- Missing authentication
- Missing rate limiting
- Improper error responses

## Language Server Integration

> **LSP Integration (agent-only):** React projects inherit TypeScript LSP integration from Node.js. See `${CLAUDE_PLUGIN_ROOT}/shared/references/lsp-integration.md` for details. Not loaded during orchestration.
