# Contributing to yata-rules

Thank you for helping improve the official Yata Cloud policy set.
This guide covers what is specific to this repository. The policy format itself is defined in the `yata-cli` design documents, which are linked below instead of repeated here (they are written in Japanese).

- [Policy evaluation](https://github.com/morryyrrom/yata-cli/blob/main/docs/architecture/Architecture-Policy-Evaluation.ja.md): rule structure, rule format, metadata, libraries, and rule tests
- [ADR: policy and fixture layout](https://github.com/morryyrrom/yata-cli/blob/main/docs/adr/ADR-0001-Policy-and-Fixture-Layout.ja.md): directory layout, rule IDs, and the fixture format
- [ADR: per-resource rule evaluation](https://github.com/morryyrrom/yata-cli/blob/main/docs/adr/ADR-0030-Per-Resource-Rule-Evaluation.ja.md): how a rule is evaluated for each resource
- [Policy engine and `yata rules test`](https://github.com/morryyrrom/yata-cli/blob/main/docs/features/Feature-0008-Policy-Engine-and-Rules-Test.ja.md): checks, exit codes, available built-in functions, and dummy values
- [Policy repository layout and distribution](https://github.com/morryyrrom/yata-cli/blob/main/docs/features/Feature-0009-Policy-Repository-Layout-and-Distribution.ja.md): the layout, versioning, release process, and CI of this repository

## Prerequisites

- `yata`, the same version as CI uses. While `YATA_CLI_REF` in [`.github/workflows/ci.yml`](.github/workflows/ci.yml) is set, build `yata-cli` from that commit (`cargo build --release --locked -p yata-cli`). Otherwise, use the `yata` release given by `min_yata_version` in [`yata-policies.json`](yata-policies.json).
- [OPA](https://github.com/open-policy-agent/opa) and [Regal](https://github.com/open-policy-agent/regal), the versions pinned in `.github/workflows/ci.yml`.

## Checking your changes locally

```sh
opa fmt --write .
regal lint .
yata rules test --policy-dir .
```

CI runs the same checks (`opa fmt --list --fail .`, `regal lint --format github .`, and `yata rules test --policy-dir . --format text`).
OPA and Regal parse Rego with the OPA implementation, while `yata` uses [regorus](https://github.com/microsoft/regorus). Only `yata rules test` shows whether a rule works with regorus, so always run it.

## Adding a rule

### Location and name

- Put the rule directly under the directory of the AWS service whose settings it checks (depth 2, no deeper directories). The directory name is the `<service>` part of the resource type (`aws-sdk-<service>`).
  - Usually this is the `<service>` of `target_type` (for example, `aws.iam.user` → `iam/`, `aws.ec2.vpc` → `ec2/`).
  - For rules whose `target_type` is `aws.account.account`, use the service of the main `related_types` (for example, CIS 4.1 → `cloudtrail/`).
  - Rules that check S3 Control (`aws.s3control.*`) go in `s3/`.
  - Exception: monitoring rules that follow the CloudTrail → CloudWatch Logs → CloudWatch → SNS chain (CIS 5.x) go in `monitoring/`.
- Name the file `<resource>_<desired state>.rego` (for example, `bucket_public_access_block`, `user_mfa_enabled`, `password_policy_min_length`). Names use `[a-z][a-z0-9_]*` and must not be Rego keywords.
- Do not put CIS control numbers in names. Put them in the `references` metadata.
- The rule ID is `yata_aws.<service>.<rule name>`, and the package is `yata.rules.yata_aws.<service>.<rule name>`.

### Format

- Follow the rule format and metadata in [Policy evaluation](https://github.com/morryyrrom/yata-cli/blob/main/docs/architecture/Architecture-Policy-Evaluation.ja.md) (the sections on rule format and writing notes).
- Do not define the same complete rule (such as `message`) with several `if` bodies. If two bodies hold for one resource, evaluation fails with a conflict. Chain the cases with `else` instead.
- Add one rule (or a small group of closely related rules) per pull request.

### Fixtures

- Put fixtures in `<rule>.fixtures/` next to the rule, named `pass_<case>.json` and `fail_<case>.json`. The format is in [ADR: policy and fixture layout](https://github.com/morryyrrom/yata-cli/blob/main/docs/adr/ADR-0001-Policy-and-Fixture-Layout.ja.md).
- Use dummy values only. `yata rules test` rejects account IDs that are not in the list of allowed dummy values ([Policy engine and `yata rules test`](https://github.com/morryyrrom/yata-cli/blob/main/docs/features/Feature-0008-Policy-Engine-and-Rules-Test.ja.md)).
- Cover every branch of `fail`, `message`, and `reason`. Built-in functions on a branch that no fixture reaches are not checked against regorus.
- Include cases with collection failure markers, missing or `null` fields, and resources in several regions.

### Built-in functions

- Use only the built-in functions that `yata` enables (the regorus cargo features in [Policy engine and `yata rules test`](https://github.com/morryyrrom/yata-cli/blob/main/docs/features/Feature-0008-Policy-Engine-and-Rules-Test.ja.md)). Some OPA built-ins are not available in regorus (for example, JWT, crypto, and `net.cidr_merge`), and OPA and Regal do not report them.
- Do not pass input values to `net.cidr_expand` or `numbers.range`. They are not stopped by the evaluation time limit.
- How rules that use `time.now_ns` are tested with fixtures is not decided yet. Discuss it in the rule's issue before writing such a rule.

## Libraries (`lib/`)

- Shared functions live in `lib/<topic>.rego`, one package per file: `yata.lib.yata_aws.<topic>`.
- Change `lib/` in its own pull request, separate from rules.
- Do not change the meaning of an existing function. Add a new function instead.

## Compatibility and the changelog

- Never move or rename a published rule file. That changes the rule ID and is a breaking change.
- When you change a rule's logic or metadata, check the impact with the versioning table in [README.md](README.md#versioning).
- Add a changelog fragment to `changelog.d/` (see [changelog.d/README.md](changelog.d/README.md)) when your pull request adds, changes, or removes a rule, changes `yata-policies.json`, or changes `lib/` in a way that changes results. CI checks for the fragment, except for changes in `lib/`, so remember those yourself.
- Do not edit `CHANGELOG.md` directly. It is updated only in release pull requests.

## Rules for new resource types

A rule that uses a resource type collected only by an unreleased `yata` cannot be tested with a released binary.
Until a `yata` release includes that collector, set `YATA_CLI_REF` in `.github/workflows/ci.yml` to a `yata-cli` commit that has it, so CI builds `yata` from source.
Keep `min_yata_version` at a released version that does not exceed the version of that commit.

## CIS Benchmarks content

Do not copy text from CIS Benchmarks (descriptions, rationale, audit or remediation procedures).
Write metadata in your own words and put the control number in `references`.
CIS Benchmarks PDFs are licensed under CC BY-NC-SA 4.0, which cannot be redistributed under this repository's license (MIT OR Apache-2.0).

## License

Contributions are provided under MIT OR Apache-2.0, as described in the Contribution section of [README.md](README.md#contribution).

## Pull requests and branches

- Follow GitHub Flow: create a feature branch from `main`, open a pull request, and merge with a merge commit.
- Name branches `feature/<issue number>-<summary>`.
- Use `<Type>: #<Issue number> <Title>` for commit messages and pull request titles (for example, `feat: #12 Add IAM user MFA rule`). Omit `#<Issue number>` when there is no issue.
