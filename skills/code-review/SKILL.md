---
name: code-review
description: Review a branch diff for logic flaws and coding standards.
---

# Code Review Skill

Use this standalone skill for `/code-review`. Review the current work against
the repository's stable base branch, attack the logic first, then check any
coding preferences discoverable in the repository. Keep the review concise and
report only useful findings.

## 1. Establish the Diff

1. Confirm the repository and current branch.
2. Choose the comparison base in this order:
   - the pull request's declared base, if reviewing a pull request;
   - `stable`, if it exists;
   - `main`, if it exists;
   - `master`, if it exists.
3. Compare the current work against that base, including committed and relevant
   uncommitted changes.
4. If no suitable base exists, say so instead of guessing.
5. Inspect the changed files and enough surrounding code, callers, tests, and
   configuration to understand their behavior.

Do not review unrelated old problems unless the change exposes or depends on
them. Do not assume a small diff is a simple diff.

## 2. Attack Logic Flaws First

For each meaningful change, identify what must remain true and try to break it.
Check only the cases relevant to the code:

- Empty, missing, malformed, duplicate, boundary, and unexpected input.
- Incorrect conditions, ranges, counts, state transitions, or data shapes.
- Error, timeout, retry, rollback, cleanup, and partial-failure paths.
- Stale state, duplicate calls, ordering, async work, and races.
- Permissions, ownership, authentication, and trust boundaries.
- Changed APIs, schemas, routes, exports, configuration, migrations, and their
  callers or consumers.
- Tests that are missing, weakened, deleted, or unable to catch the regression.

For security-sensitive, persistent, concurrent, numeric, or external-service
changes, trace at least one concrete abuse or failure path. Do not label
something a race, security flaw, or validation bug without showing the path.

A logic finding must explain:

- **Location:** file and line or an unambiguous code location.
- **Failure:** the input or execution sequence that breaks the behavior.
- **Impact:** what becomes incorrect, unavailable, insecure, or corrupted.
- **Fix:** the smallest sensible correction or regression test.

Do not report vague risks, generic best practices, speculative optimisations, or
style issues as logic defects. Re-check guards, callers, framework behavior, and
tests before reporting a finding. Merge duplicate symptoms into one root-cause
finding.

## 3. Check Coding Preferences

Only after the logic review:

1. Read repository-local instructions and any coding-preference files or
   directories that the user or repository identifies.
2. Apply only rules relevant to the changed code.
3. If no applicable preference is discoverable, say so and do not invent one.
4. Keep preference violations separate from logic findings.
5. Treat style as non-blocking unless the preference is an explicit build, CI,
   compatibility, or correctness requirement.

## 4. Verify and Report

Run the narrowest relevant tests, lint, type checks, build, or focused
reproduction that the repository documents. Record real results. Distinguish
`PASS`, `FAIL`, `BLOCKED`, and `NOT RUN`; never claim a check passed without
running it.

Do not modify the reviewed code unless the user explicitly asks for fixes.

Use this compact output:

```markdown
## Findings

### [blocking] `path/to/file:42` — short title

- **Failure:** Concrete broken path.
- **Impact:** Why it matters.
- **Fix:** Focused correction or test.

## Coding preferences

- Sources: [paths, or “none found”]
- Adherence: [problems, or “adheres”]

## Verification

- `[command]` — PASS/FAIL/BLOCKED/NOT RUN: result

## Summary

[Safe to merge, or the most important issue preventing that.]
```

Use `blocking` only for a demonstrated correctness, security, data-loss, or
release-stopping issue. Use `major` or `minor` for lower-impact concrete bugs.
Use `note` for optional improvements. If no real defect is found, say **“No
confirmed logic findings.”** Keep the report short.

## Rules of Restraint

- Logic before style.
- Stable-base diff before broad repository review.
- Concrete evidence before criticism.
- Relevant preferences only.
- No invented standards.
- No silent fixes.
- No fabricated test results.
