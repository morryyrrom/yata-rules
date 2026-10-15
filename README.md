# yata-rules

The official AWS policy set for [Yata Cloud](https://github.com/morryyrrom/yata-cli).
It contains rules written in Rego, based on the CIS AWS Foundations Benchmark and similar guidance, that `yata` evaluates against the configuration it collects from an AWS account.

This repository is for users of `yata`. The `yata` binary does not include these rules; you get them from here and point `yata` at them.

## Status

This policy set is in early development. There is no release yet, and rule IDs may change before 1.0.0 (see [Versioning](#versioning)).

## Compatibility

The oldest `yata` version that can load this policy set is `min_yata_version` in [`yata-policies.json`](yata-policies.json).
Each release note also states the required version (for example, `Requires yata >= 0.1.0.`).

If your `yata` is too old, or does not support the manifest's `format_version`, `yata` stops with exit code 2 before evaluating anything.
Update `yata`, or use an older release of this policy set.

## Getting the policies

The official distribution is the archive attached to each [GitHub Release](https://github.com/morryyrrom/yata-rules/releases).

### Release archive (recommended)

```sh
VERSION=0.1.0
BASE=https://github.com/morryyrrom/yata-rules/releases/download/v${VERSION}
curl -fLO "${BASE}/yata-rules-${VERSION}.tar.gz"
curl -fLO "${BASE}/SHA256SUMS"
sha256sum --check --ignore-missing SHA256SUMS
tar -xzf "yata-rules-${VERSION}.tar.gz"
# Optional: verify that the asset belongs to the release (GitHub CLI)
gh release verify-asset "v${VERSION}" "yata-rules-${VERSION}.tar.gz" --repo morryyrrom/yata-rules
```

- On macOS, use `shasum -a 256 --check --ignore-missing SHA256SUMS` instead of `sha256sum`.
- On Windows, use `yata-rules-<version>.zip` and compare the output of `Get-FileHash -Algorithm SHA256` with `SHA256SUMS`.
- Do not use the "Source code" archives that GitHub adds to every release automatically. They cannot be checked with `SHA256SUMS` or `gh release verify-asset`.

`yata` does not verify the signature or origin of policies. Verifying them is up to you, using `SHA256SUMS` and the release attestation shown above.

### git clone

```sh
git clone --branch v0.1.0 --depth 1 https://github.com/morryyrrom/yata-rules.git
```

Always check out a release tag. Do not use `main`:

- `main` contains unreleased rules, which may need a `yata` version that is not released yet (exit code 2).
- The `version` in `yata-policies.json` on `main` is the previous release, so it does not match the content.

To update, run `git fetch --tags` and `git checkout v<version>`.

## Usage

Pass the directory to `yata` with `--policy-dir`, or set `YATA_POLICY_DIR`:

```sh
yata scan --policy-dir ./yata-rules-0.1.0
# or
export YATA_POLICY_DIR="$PWD/yata-rules-0.1.0"
yata scan
```

There is no default location. If neither is given, `yata` stops with exit code 2.
`yata` never downloads or updates policies by itself.

### Listing the rules

```sh
yata rules list --policy-dir ./yata-rules-0.1.0
```

This README does not keep a list of rules; use the command above.

## Updating

1. Read [CHANGELOG.md](CHANGELOG.md) for the new version. Check for removed or renamed rule IDs (breaking changes) and for a new `min_yata_version`.
2. Extract the new version into a separate directory.
3. Switch `--policy-dir` or `YATA_POLICY_DIR` to it. If something goes wrong, switch back to the previous directory.

## Your own rules

- Do not add files to the official policy set directory. They are lost when you update, and the reported `policies[].version` no longer matches the content.
- Put your own rules in a separate directory with its own manifest and a different `name` (for example, `my_policies`), and pass `--policy-dir` more than once.
- If you fork and modify this policy set, change `name` in `yata-policies.json`. Otherwise reports cannot tell your modified set from the official `yata_aws` set.

## Versioning

`version` follows [Semantic Versioning 2.0.0](https://semver.org/spec/v2.0.0.html) without pre-release parts (`x.y.z` only).
A change is breaking when you cannot keep using the same settings (`--rule` selections, suppressions by rule ID, SARIF alert tracking, comparing results) with the same `yata` after updating.

| Change | From 1.0.0 | While 0.y.z |
| --- | --- | --- |
| Removing a rule, changing a rule ID, changing a rule's `target_type`, raising `format_version` | major | y |
| Adding a rule, raising `min_yata_version`, changing what a rule checks, changing `severity` or `tags` | minor | z |
| Fixing a wrong result (including new `fail` results), changing `message`, `reason`, or other metadata wording, changes in `lib/` that do not change any result | patch | z |

When a release contains several kinds of changes, the largest one decides the version.
Only the latest release line is maintained.
The full policy is in the design document [Policy repository layout and distribution](https://github.com/morryyrrom/yata-cli/blob/main/docs/features/Feature-0009-Policy-Repository-Layout-and-Distribution.ja.md) (Japanese).

## About the benchmarks

Rules refer to CIS Benchmarks control numbers in their metadata (`references`).
This project is not endorsed or certified by the Center for Internet Security (CIS). The rule descriptions are written independently and do not reproduce the benchmark text.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

Licensed under either of

- Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE) or <https://www.apache.org/licenses/LICENSE-2.0>)
- MIT license ([LICENSE-MIT](LICENSE-MIT) or <https://opensource.org/licenses/MIT>)

at your option.

### Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted for inclusion in the work by you, as defined in the Apache-2.0 license, shall be dual licensed as above, without any additional terms or conditions.
