# Technical Debt Review - Example Output

Sample findings from a technical debt review showing various debt types and fix formats.

---

## Example 1: Deprecated Dependency (prompt fix)

```yaml
issues:
  - title: "Deprecated dependency: request package"
    file: "package.json"
    line: 15
    category: "Technical Debt"
    severity: "Major"
    description: |
      The 'request' package has been deprecated since February 2020 and receives
      no security updates. It has known vulnerabilities and should be replaced.
    debt_type: "deprecated_dependency"
    urgency: "soon"
    effort_estimate: "medium"
    fix_type: "prompt"
    fix_prompt: |
      Replace 'request' package with 'node-fetch' or 'axios':
      1. Install replacement: npm install node-fetch
      2. Find all usages: grep -r "require('request')" src/
      3. Update each usage to use fetch API or axios equivalent
      4. Update tests that mock the request module
      5. Remove request from package.json
```

---

## Example 2: Outdated Pattern - Class Component (prompt fix)

```yaml
issues:
  - title: "React class component in React 18+ project"
    file: "src/components/UserProfile.tsx"
    line: 1
    range: "1-95"
    category: "Technical Debt"
    severity: "Minor"
    description: |
      Class component pattern in a React 18 project. Functional components
      with hooks are the modern standard and offer better performance,
      simpler testing, and improved code reuse through custom hooks.
    debt_type: "outdated_pattern"
    urgency: "low"
    effort_estimate: "small"
    fix_type: "prompt"
    fix_prompt: |
      Convert UserProfile from class to functional component:
      1. Replace class declaration with function component
      2. Convert this.state to useState hooks
      3. Convert componentDidMount to useEffect with empty deps
      4. Convert componentDidUpdate to useEffect with dependencies
      5. Convert componentWillUnmount cleanup to useEffect return function
      6. Remove 'this.' references, use local variables
      7. Update any refs from createRef to useRef
```

---

## Example 3: Workaround with HACK Comment (prompt fix)

```yaml
issues:
  - title: "HACK comment indicating workaround"
    file: "src/api/client.ts"
    line: 123
    range: "123-135"
    category: "Technical Debt"
    severity: "Major"
    description: |
      Explicit HACK comment indicating a workaround for a library bug.
      The referenced bug (axios#4567) was fixed in axios 1.2.0, but the
      project is still on 0.27.x with this workaround.
    debt_type: "workaround"
    urgency: "soon"
    effort_estimate: "small"
    fix_type: "prompt"
    fix_prompt: |
      Remove workaround and upgrade axios:
      1. Upgrade axios to 1.2.0+: npm install axios@latest
      2. Remove the HACK block at lines 123-135
      3. Test the affected API calls to verify fix works
      4. Update any axios type imports if needed
```
