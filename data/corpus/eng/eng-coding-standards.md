---
doc_id: eng-coding-standards
title: Coding Standards
folder: eng
sensitivity: internal
---

# Coding Standards

These standards apply to all production code. Linters enforce most of them; reviewers enforce the rest.

## General

- **Readable over clever.** Code is read far more often than it is written.
- Every service has a `README.md` with how to run it, test it and deploy it.
- Keep PRs under **400 changed lines** where possible. Large PRs need a short design note.
- Every PR needs at least **one approval**. Changes to auth, tenancy, billing or storage permissions need **two**, one from the owning team.

## Go (Beacon services)

- Format with `gofmt`; lint with `golangci-lint` using the shared config.
- Return errors, don't panic. Wrap errors with context: `fmt.Errorf("load tenant %s: %w", id, err)`.
- Pass `context.Context` as the first argument of anything that does I/O.
- No global mutable state.

## TypeScript (Ledger services and web)

- `strict: true` in `tsconfig`. No `any` without a comment explaining why.
- Format with Prettier, lint with ESLint (shared config).
- Validate all external input with **zod** schemas at the boundary.
- Money: integers in minor units, never `number` floats with decimals. Use the shared `Money` type.

## Tenancy

- Every database query on tenant data must go through the tenancy library, which adds the `tenant_id` filter.
- Writing raw SQL that touches tenant tables requires a comment `// tenancy-reviewed: <reason>` and a second reviewer.

## Testing

- Unit tests for business logic; integration tests against real dependencies via Docker.
- Minimum **80% line coverage** on new code in billing (`ledger-invoicer`, `ledger-payments`).
- Bug fixes come with a test that would have caught the bug.
- Flaky tests are quarantined within one day and fixed within one week.

## Logging

- Structured JSON logs through the shared logging library.
- **Never log** secrets, API keys, passwords, full payment card numbers or authentication tokens. The logging library redacts known fields, but don't rely on it.
- Include `tenant_id` and `request_id` in every log line.

## Dependencies

- New third-party dependencies need a short justification in the PR.
- Dependabot PRs are merged weekly by the owning team.
- No dependencies with GPL or AGPL licences in shipped code.

## Secrets

Never commit secrets. The pre-commit **gitleaks** hook blocks most of them. See *Secrets Management*.
