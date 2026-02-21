# Common Bug Patterns

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

## Array/Promise Pitfalls

Promise.allSettled vs Promise.all: using Promise.all when partial failure is acceptable causes cascade failures. Severity: Major.
Array.at(-1) vs arr[arr.length-1]: latter returns undefined from -1 index on empty arrays. Severity: Minor.

## .NET Specific

IDisposable not disposed — connection/stream leak; use `using var`. Severity: Major.
Async deadlock — `.Result`/`.Wait()` in ASP.NET synchronization context; use `await`. Severity: Critical.
