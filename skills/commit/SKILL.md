---
name: commit
description: Use consistent Conventional Commit messages.
---

# Git Commit Convention Skill

Use this standalone skill when creating, reviewing, or correcting Git commit
messages. It contains the complete convention and does not depend on access to
any repository, preference file, or external documentation.

## When to Use

- Before creating a Git commit.
- When reviewing a proposed commit message.
- When correcting a commit message that does not follow the convention.
- When deciding whether a change should be split into separate commits.

## Commit Format

```text
<type>[optional scope][!]: <description>

[optional body]

[optional footer(s)]
```

A one-line commit is preferred when the subject provides enough context.

## Subject Line

- Use a recognised, lowercase type.
- Add a short, lowercase scope when it provides useful context.
- Put `!` before the colon for a breaking change.
- Use the imperative mood: `add`, `fix`, `remove`, or `update`.
- Aim for 50 characters or fewer; never exceed 72 characters.
- Do not end the description with a period.
- Do not include an issue number unless the project explicitly requires it.
- The subject must complete this sentence naturally: “If applied, this commit
  will <description>.”

Good:

```text
docs: clarify installation steps
fix(auth): reject expired sessions
refactor: simplify event registration
```

Avoid:

```text
Fixed the auth bug
Updating docs.
More changes
```

## Types

Use the most specific applicable type:

| Type | Use for |
|---|---|
| `feat` | A new user-facing or developer-facing feature |
| `fix` | A bug fix |
| `docs` | Documentation-only changes |
| `refactor` | Code restructuring without a behaviour change |
| `test` | Adding or changing tests |
| `perf` | A performance improvement |
| `style` | Formatting or stylistic changes with no behaviour change |
| `build` | Build system or dependency changes |
| `ci` | Continuous integration or deployment changes |
| `chore` | Maintenance that does not fit another type |
| `revert` | Reverting an earlier commit |

These are conventions, not an exhaustive restriction. Choose a clear,
consistent type if the project needs an additional one.

## Scope

A scope is optional and identifies the affected part of the codebase:

```text
fix(parser): handle empty input
ci(github-actions): cache npm dependencies
```

Use a noun or short component name. Do not force a scope when it adds no
information.

## Body

Add a body when the subject alone does not provide enough context.

- Separate the subject and body with one blank line.
- Explain why the change is needed, including motivation, constraints, or
  important consequences.
- Focus on why rather than how; the diff already shows most of how.
- Wrap body lines at approximately 72 characters.
- Use additional paragraphs or bullets when they improve readability.

```text
fix(cache): preserve entries during refresh

Refreshing the cache previously replaced the entire map before the new
entries had been validated. Keep the existing entries until validation
succeeds so a failed refresh cannot discard usable data.
```

## Breaking Changes and Footers

Mark a breaking change either with `!` in the header or with a footer:

```text
feat(api)!: rename the user endpoint

BREAKING CHANGE: clients must call `/users` instead of `/account`.
```

Other trailers may be used when the project has an agreed convention:

```text
fix: prevent duplicate notifications

Refs: #123
Reviewed-by: Name
```

Put footers after a blank line following the body. Use uppercase
`BREAKING CHANGE` when it appears as a footer token.

## Commit Procedure

1. Inspect the final staged diff before choosing the message.
2. Identify the most specific applicable type.
3. Add a concise scope only when it adds information.
4. Write an imperative subject without a trailing period.
5. Mark breaking changes with `!` or a `BREAKING CHANGE` footer.
6. Add a body only when the subject cannot explain the important context.
7. Keep one logical change per commit and exclude unrelated formatting changes.
8. Confirm that the message describes the final staged diff, not the original
   plan.

Completion criterion: the staged diff is accounted for, and the message's type,
scope, subject, body, and breaking-change marker accurately describe it.

## Verification

Before finalising a commit message, check:

- The type is lowercase and appropriate.
- The scope is short, lowercase, and useful, or omitted deliberately.
- The subject is imperative, concise, and has no final period.
- The subject is no longer than 72 characters.
- The body and footers are separated by blank lines when present.
- Any breaking change is explicitly marked.
- The message describes one logical change.

## Sources

- [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/)
- [How to Write a Git Commit Message](https://cbea.ms/git-commit/)
