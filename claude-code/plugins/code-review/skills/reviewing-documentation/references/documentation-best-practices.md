# Documentation Best Practices Reference

## README.md Required Sections

1. **Title/Description** - name, one-paragraph description, key features
2. **Installation** - prerequisites with versions, step-by-step, verification
3. **Quick Start** - minimal working example, expected output
4. **Configuration** - required env vars, config format, defaults
5. **API/CLI Reference** - commands/methods, parameters, return values
6. **Contributing** - bug reports, PR process, code style
7. **License**

**Anti-patterns**: assuming global deps, non-working examples, missing prerequisites, outdated screenshots

## Severity Calibration

Missing Install section: Critical (blocks users). Missing error responses in API docs: Major. Missing Contributing: Suggestion (not blocking).

## API Documentation

Required per endpoint: all params with types, all response codes, at least one example, error responses.

## Consistency Patterns

One term per concept (not "config/conf/settings"). Consistent heading case, list punctuation, code language tags, link style throughout.
