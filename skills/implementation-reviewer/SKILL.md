---
name: implementation-reviewer
description: >
  Critique a feature implementation for common engineering pitfalls: obsolete code,
  dead code, duplicate logic, unnecessary backward compatibility, erroneous fallbacks,
  misaligned defaults, stale documentation, and phantom references. Use when the user
  asks to review, audit, critique, or find problems in a recently implemented feature,
  or asks "what's wrong with this implementation?" or "find issues in this code."
  Do NOT use for adding new features or general code style review.
---

## Review Process

1. Read the implementation: all modified/added files, the tests, and any documentation that references the feature.
2. Run the checklist below against each file. Track findings as `[file:line] category — description`.
3. Report findings grouped by severity (blockers first, then warnings, then nits).

## Checklist

### Obsolete code
- Functions, fields, or constants that existed before the feature but are no longer reachable after it. Grep for all call sites; if zero remain, flag it.
- Feature flags, config fields, or enum variants introduced for migration that were never removed after the migration completed.
- Imports that are no longer used after the refactor.

### Dead code paths
- Branches guarded by conditions that can never be true given the new logic (e.g., `if trigger == "manual"` after the `trigger` field was removed).
- `except` handlers that catch exceptions the new code can no longer raise.
- Default parameter values that no caller ever exercises.

### Duplicate logic
- The same operation implemented in two places (e.g., a helper and an inline copy). Search for similar function signatures, identical string literals, and repeated multi-line blocks.
- Two methods that do nearly the same thing with minor variations — candidates for consolidation.

### Unnecessary backward compatibility
- Shims, aliases, or re-exports kept "for backward compatibility" when no external consumer exists (internal project, no published API).
- Fallback code paths that silently degrade to old behavior instead of failing fast, masking bugs.
- Renamed-but-kept-old-name patterns (`old_func = new_func`) with no deprecation timeline.

### Erroneous fallback methods
- Fallbacks whose assumptions no longer hold (e.g., falling back to a global namespace after introducing per-instance namespaces).
- `or` / `if not` chains where the fallback value contradicts the new design intent.
- Default branches in routing/dispatch that swallow unexpected states instead of raising.

### Misaligned defaults
- Config defaults that contradict the feature's intended behavior (e.g., a progressive-disclosure feature defaulting to `False`).
- Constructor defaults that differ from config defaults for the same setting.
- Environment variable overrides that are missing for newly added config fields.

### Stale documentation
- CLAUDE.md, README, or docstrings that describe removed behavior (e.g., referencing a deleted `trigger` field).
- Code comments that describe "current" behavior that is now the old behavior.
- Roadmap or TODO items that were completed but not marked done.
- Test names or test docstrings that no longer match what the test actually verifies.

### Phantom references
- Tests that assert on entities (files, functions, fields) that don't exist in the current code — they pass only because they're mocked or never actually exercised.
- Documentation citing a glob pattern, file path, or API that was changed but the docs weren't updated.
- Prompt templates referencing format slots that are no longer populated.

### Test gaps
- New public methods or config fields with no corresponding test.
- Existing tests that should have been updated to reflect the new behavior but still test the old contract.
- Tests that mock so aggressively they'd pass even if the feature were deleted.

## Report Format

```
## [Feature Name] Implementation Review

### Blockers
- [file:line] **category** — description

### Warnings
- [file:line] **category** — description

### Nits
- [file:line] **category** — description

### Summary
N blockers, N warnings, N nits across N files.
```

Omit empty severity sections. Keep descriptions to one sentence — link to the line, not to a paragraph of explanation.
