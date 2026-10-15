# Changelog fragments

Each pull request that changes what users get records its change here as a separate file (a fragment).
Release pull requests collect the fragments into [CHANGELOG.md](../CHANGELOG.md) and delete them.
Separate files keep parallel pull requests from conflicting in `CHANGELOG.md`.

This README is not a fragment and is never deleted.

## When a fragment is needed

Add exactly one fragment to a pull request that:

- adds, changes, or removes a rule (a `.rego` file outside `lib/`),
- changes `yata-policies.json`, or
- changes `lib/` in a way that changes the result of any rule.

Pull requests that change only fixtures, CI, or documentation, and `lib/` changes that do not change any result, do not need one.
CI checks the first two cases. A maintainer can add the `skip-changelog` label to a pull request that does not need a fragment (for example, a formatting-only change to a rule).

## File name

`changelog.d/<issue number>-<summary>.md`, for example `changelog.d/12-iam-user-mfa.md`.

- `<summary>` uses lowercase letters, digits, and `-` only.
- Without an issue, use `<summary>.md`.

The file name must match `^([0-9]+-)?[a-z0-9]+(-[a-z0-9]+)*\.md$`.

## Content

Use the [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) section headings you need, each followed by at least one bullet:

- `### Added`
- `### Changed`
- `### Deprecated`
- `### Removed`
- `### Fixed`
- `### Security`

Other headings (`#`, `##`, or other `###` titles) are not allowed.

- Start each bullet about a rule with its rule ID.
- When a rule ID changes, write both the old and the new ID.
- When a fix can produce new `fail` results, say so under `### Fixed`.
- When `min_yata_version` is raised, write `Requires yata >= <version>.` under `### Changed`.

Example:

```markdown
### Added

- `yata_aws.iam.user_mfa_enabled`: IAM users with a console password must have MFA enabled.
```
