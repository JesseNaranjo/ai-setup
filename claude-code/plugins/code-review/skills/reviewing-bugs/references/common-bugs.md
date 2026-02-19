# Common Bug Patterns

## Null/Undefined Reference

Missing optional chaining on nested access. Severity: Major (crashes), Minor (undefined behavior).

## Off-by-One Errors

`i <= arr.length` (out of bounds), wrong split count. Severity: Minor to Major.

## Async/Promise Issues

Unhandled promise rejection — async call without `await` or `.catch()`. Severity: Major.
Floating promises — missing `await`, shows success before operation completes. Severity: Major.

## Race Conditions

State changed between awaits — atomic fix: `db.query('UPDATE users SET count = count + 1 WHERE id = ?', [id])`. Severity: Major to Critical.

## Type Coercion

`==` vs `===`, string concat not addition (`'10' + 5`), lexicographic `.sort()`. Severity: Minor to Major.

## State Management

Mutating shared state — `items.sort()` mutates original; use `[...items].sort()`. Severity: Major.
Stale closure — React `useEffect` with empty deps captures initial value; add dependency or use ref. Severity: Major.

## Error Handling

Empty catch blocks silently swallowing errors. Overly broad catch hiding real failures. Severity: Minor to Major.

## .NET Specific

IDisposable not disposed — connection/stream leak; use `using var`. Severity: Major.
Async deadlock — `.Result`/`.Wait()` in ASP.NET synchronization context; use `await`. Severity: Critical.
