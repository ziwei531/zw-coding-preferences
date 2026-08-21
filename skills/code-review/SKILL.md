---
name: code-review
description: Attack logic flaws, then review repository coding standards.
---

# Code Review Skill

Use this standalone skill for reviewing a patch, working-tree change, commit
range, branch, pull request, or proposed implementation. The review is
correctness-first: actively try to break the change, validate every candidate
finding, then review applicable repository and user coding standards. Do not
depend on access to this repository or to any particular preference file.

## Core Policy

- Review behavior before style.
- Seek high recall during analysis, but report only high-precision findings.
- Separate **severity** from **confidence**.
- Do not block on a speculative concern.
- Do not invent repository standards.
- Do not silently modify the code during a review unless the user asks for fixes.
- Never claim a test, build, scan, or reproduction passed without real output.

A beautifully styled bug is still a bug. A vague concern is not a finding.

## Mandatory Review Order

1. Establish scope and review the complete change.
2. Build a change map and identify affected contracts.
3. Adversarially attack correctness and security.
4. Check completeness and cross-file integration.
5. Validate, deduplicate, and prioritise candidate findings.
6. Review repository and user coding standards.
7. Run targeted verification.
8. Report findings, uncertainty, standards, and verification status.

Do not begin with naming, formatting, or personal preference.

## Phase 1: Establish Scope

Identify the review input and its boundaries:

- Working-tree or staged diff.
- Commit or commit range.
- Branch comparison, including base and head.
- Pull request.
- Individual files or an implementation proposal.

Before reasoning about the change:

1. Confirm the base, head, and repository state.
2. List changed files and classify them as source, test, configuration,
   migration, dependency, generated, vendor, documentation, or deployment.
3. Inspect surrounding code, callers, consumers, tests, schemas, configuration,
   and relevant history when needed.
4. Identify generated or vendor files and state whether they were skipped.
5. For a large diff, split the review by logical change, then perform a final
   integration pass across the whole change.
6. Record unavailable context instead of silently assuming it.

A diff alone is not execution context. A small diff is not automatically a safe
diff.

## Phase 2: Build the Change Map

Summarise the change in terms of behavior, not file names:

- What user, system, or data behavior is intended to change?
- What invariant must remain true?
- Which inputs, state transitions, outputs, and side effects are involved?
- Which public APIs, schemas, storage formats, jobs, routes, events, or consumers
  are affected?
- What failure behavior is expected?
- What permissions, trust boundaries, dependencies, or operational assumptions
  are involved?

Then identify the proof obligations: the specific facts that must be true for
the change to be correct. Attack those obligations directly.

## Phase 3: Adversarial Logic Attack

Try to make the change fail. Trace concrete inputs and execution sequences
through the implementation rather than trusting names, comments, or intent.

### General attack set

Check applicable cases:

- Empty, null, malformed, duplicated, oversized, and unexpected input.
- Zero, one, minimum, maximum, negative, and boundary values.
- Missing, stale, expired, partially written, or corrupted state.
- Repeated calls, retries, replays, duplicate events, and idempotency.
- Time zones, daylight-saving changes, locale, encoding, precision, rounding,
  and date boundaries.
- Reordering, partial failure, timeout, cancellation, and retry behavior.
- Concurrent calls, stale responses, shared mutable state, and lost updates.
- Permission, identity, tenant, ownership, and unauthenticated paths.
- Resource exhaustion, unbounded loops, recursion, pagination, and file growth.
- Error paths, fallback paths, cleanup, rollback, and observability.
- External service, browser, filesystem, environment, and dependency behavior.

For each relevant attack:

1. State the expected invariant or contract.
2. Construct a hostile input or execution sequence.
3. Trace the actual path, including guards and downstream effects.
4. Decide whether the invariant survives.
5. Check callers, tests, and framework behavior before reporting a defect.

### Change-specific attack modules

Select modules from the change map; do not run irrelevant checklists.

**API, UI, or input changes**

- Test malformed, missing, repeated, unauthorized, and oversized input.
- Check validation at the real trust boundary, not merely at a convenient caller.
- Check response shape, status, error leakage, compatibility, and client impact.

**Persistence, schema, or migration changes**

- Check old data, new data, nullability, defaults, indexes, rollback, and
  partial migration states.
- Confirm readers and writers agree on the new shape.
- Confirm deployment ordering will not create a mixed-version outage.

**Async, concurrency, or distributed changes**

- Construct an explicit interleaving, not a generic “race condition” claim.
- Check cancellation, retries, timeouts, ordering, deduplication, locks, and
  stale writes.
- Check whether failure of one worker or service leaves durable inconsistent
  state.

**Security-sensitive changes**

- Identify assets, actors, trust boundaries, attacker-controlled sources, and
  sensitive sinks.
- Trace data flow to authorization, HTML, SQL, shell, filesystem, redirects,
  templates, deserialization, logging, crypto, and network boundaries.
- Check abuse cases and business-logic bypasses, not only dangerous-function
  patterns.

**Dependency, build, or deployment changes**

- Check transitive impact, lockfiles, version compatibility, reproducibility,
  permissions, secrets, artifact paths, and rollback.
- Look for dependency confusion, typosquatting, untrusted install scripts, and
  configuration that differs between local, CI, and production environments.

**Frontend or browser changes**

- Check loading, refresh, history, navigation, accessibility, keyboard use,
  responsive states, stale async responses, XSS, and asset/base-path behavior.

**Randomness, probability, financial, or numeric changes**

- Check the sample space, layered probabilities, rounding, precision, seeding,
  independence assumptions, and whether labels match the implemented event.
- Compare a focused result with a theoretical or invariant-based expectation
  when one is available.

**Performance or resource changes**

- Check input-dependent complexity, unbounded work, memory, connection/file
  lifetime, cache growth, and behavior under realistic worst cases.

## Phase 4: Completeness and Integration

Ask what must change alongside the edited lines. Common omissions include:

- A migration without compatible readers or rollback.
- New behavior without route, command, export, registration, or feature wiring.
- Changed schema or API without updated consumers, fixtures, or documentation.
- Changed configuration without validation, defaults, or deployment updates.
- New failure mode without error handling, logging, metrics, or user feedback.
- Behavior change without a regression test or meaningful update to an existing
  test.
- Deleted or weakened tests that no longer fail when the behavior breaks.
- Dependency update without lockfile, license, compatibility, or security review.
- Generated output that no longer matches its source.

This phase is not permission to demand unrelated refactors. Check only the
pieces required by the change map and its contracts.

## Phase 5: Validate Candidate Findings

Do not report a candidate immediately. For each candidate:

1. Re-open the exact code location.
2. Demonstrate the triggering input or execution sequence.
3. Check whether a caller, middleware, type system, framework, test, or later
   guard already prevents the failure.
4. Check whether the behavior is intentional and documented.
5. Estimate reachability, affected scope, and impact.
6. Suggest the smallest focused fix or regression test.
7. Discard duplicates and merge manifestations caused by one root cause.

A candidate is reportable only if it has a concrete failure path, a material
missing proof, or a clearly stated project requirement that it violates.

### Confidence and severity are separate

Use confidence to describe how strongly the evidence supports the finding:

- `high`: directly demonstrated or mechanically verified.
- `medium`: a credible path exists, but a relevant assumption or environment
  detail is not fully confirmed.
- `low`: plausible hypothesis requiring clarification or reproduction.

Use severity to describe impact if the finding is true:

- `blocker`: demonstrated correctness, security, data-loss, or release-stopping
  defect.
- `major`: likely important-path failure, contract break, or missing required
  handling that should be fixed before merge.
- `minor`: contained defect or robustness gap.
- `note`: non-blocking improvement, trade-off, or educational observation.

Never turn a low-confidence high-impact hypothesis into a proven blocker. Report
it as a question or escalation with the missing evidence.

### Finding statuses

When useful, preserve uncertainty explicitly:

- `confirmed`: concrete failure demonstrated.
- `needs-context`: potentially important, but repository or runtime context is
  missing.
- `acceptable-risk`: understood trade-off that does not require a fix.
- `false-positive`: guard, contract, or framework behavior invalidates it.
- `duplicate`: same root cause as another finding.

Do not hide a rejected candidate in the final findings list.

## Phase 6: Repository Coding Standards

Only after logic, completeness, and candidate validation, discover standards
for the changed code.

Search in this order:

1. Repository-local agent or contributor instructions.
2. User-mentioned coding preferences or preference directories.
3. Project documentation defining implementation conventions.
4. Formatter, linter, type-checker, and build configuration.
5. Language or framework defaults only when no project-specific rule exists.

Use the repository's actual names and locations. Do not assume a language,
filename, directory, or preference document. Apply only standards relevant to
the changed files.

Review applicable conventions such as naming, formatting, imports, API shape,
error handling, tests, dependencies, architecture, comments, and documentation.
A style preference is non-blocking unless it is explicitly a compatibility,
correctness, build, CI, or project requirement.

If no applicable standards are discoverable, say **“No repository-specific
coding standards found.”** Do not manufacture generic violations.

## Phase 7: Verification

Run the narrowest meaningful checks after analysis:

- Existing tests covering changed behavior.
- Focused regression tests for demonstrated edge cases.
- Formatter, linter, type checker, build, security, dependency, or schema checks
  configured by the project.
- A minimal reproduction for a reported logic flaw when practical.

Use documented project commands first. Record the exact command and result as
`PASS`, `FAIL`, `BLOCKED`, or `NOT APPLICABLE`.

Distinguish environmental failures from regressions. Treat tools as evidence,
not proof: a green test suite does not invalidate a demonstrated logic flaw,
and an automated alert is not a confirmed vulnerability until its data flow and
context are checked.

Never silently change the patch to make verification pass. If the user asks for
a fix, apply it separately and repeat the review from the changed diff.

## Review Output

Be concise and prioritised. Findings come before advisory material.

```markdown
## Findings

### [blocker] `path/to/file.ext:42` — concise title

- **Confidence:** high
- **Status:** confirmed
- **Failure:** Concrete input or execution sequence that breaks the invariant.
- **Impact:** User, data, security, reliability, or operational consequence.
- **Evidence:** Relevant guard, caller, test result, or reproduction.
- **Fix:** Smallest focused correction and/or regression test.

## Standards review

- Sources discovered: [paths or “none”]
- Applicable rules: [rules or “none”]
- Violations: [findings or “none”]

## Verification

- `[command]` — PASS/FAIL/BLOCKED/NOT APPLICABLE: result

## Coverage and residual risk

- Reviewed: [scope, files, and dimensions]
- Skipped: [generated/vendor/unavailable context, if any]
- Residual risk: [short statement or “none identified”]

## Summary

[Whether the change is safe to merge, with the reason in one or two sentences.]
```

If there are no reportable findings, say **“No confirmed findings.”** Still
report standards discovered, checks run, skipped scope, and meaningful residual
risk. Keep optional suggestions separate from merge-blocking findings.

## Pitfalls

- Reviewing only the diff and missing callers, consumers, state, or deployment
  ordering.
- Running every checklist regardless of the change and generating noise.
- Calling a race, validation flaw, or security issue without a concrete path.
- Treating a pattern match, test presence, or green CI as proof of correctness.
- Confusing severity with confidence.
- Reporting the same root cause at multiple locations.
- Blocking on subjective refactors, generic best practices, or educational nits.
- Assuming a preference file exists or naming one without discovering it.
- Silently fixing code when the user requested review only.
- Claiming verification from commands that were not actually run.

## Research Basis

The workflow is informed by public review guidance from:

- [CodeRabbit documentation](https://docs.coderabbit.ai/)
- [Cursor Bugbot documentation](https://cursor.com/docs/bugbot)
- [GitHub Copilot code review documentation](https://docs.github.com/en/copilot/how-tos/copilot-on-github/use-copilot-agents/copilot-code-review)
- [Google Engineering Practices](https://google.github.io/eng-practices/review/reviewer/standard.html)
- [OWASP Secure Code Review Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secure_Code_Review_Cheat_Sheet.html)
- [Semgrep false-positive guidance](https://docs.semgrep.dev/kb/semgrep-code/reduce-false-positives)
