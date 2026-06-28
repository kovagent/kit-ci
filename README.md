# kit-ci

Shared CI for the `*kit` Rust data libraries (`sectorkit`, `curvekit`,
`indexkit`, `divkit`, `hourskit`). Reusable GitHub Actions workflows plus the
canonical hygiene-file templates, so every kit stays aligned without a
monorepo. Each kit keeps its own repo, history, release cadence, and nightly
data commits.

## Reusable workflows

| Workflow | What it does | Caller trigger |
|---|---|---|
| `ci.yml` | fmt, clippy `-D warnings`, tests, doc-tests, rustdoc `-D warnings`, cargo-deny, cargo-audit; optional MSRV + manifest-drift jobs | `push` / `pull_request` |
| `release.yml` | full CI gate, then `cargo publish` each crate in dependency order | `push` tag `v[0-9]*` |
| `security-nightly.yml` | daily advisory-db sweep (audit + deny), opens a tracking issue on a finding | `schedule` |

## Wiring a kit

Pin `uses:` to a full kit-ci commit SHA, not a branch or tag, so a change to
kit-ci cannot alter a kit's CI or a tagged release. Replace `<commit-sha>`
below with the SHA you are pinning to.

`.github/workflows/ci.yml`:

```yaml
name: CI
on:
  push: { branches: [main] }
  pull_request: { branches: [main] }
jobs:
  ci:
    uses: userFRM/kit-ci/.github/workflows/ci.yml@<commit-sha>
    with:
      msrv: "1.86"                 # omit to skip the MSRV job
      manifest_check: |            # omit to skip; fail on data drift
        cargo run --bin indexkit-cli -- manifest
        git diff --exit-code data/manifest.json
```

`.github/workflows/release.yml`:

```yaml
name: Release
on: { push: { tags: ["v[0-9]*"] } }
jobs:
  release:
    uses: userFRM/kit-ci/.github/workflows/release.yml@<commit-sha>
    with: { crates: "indexkit indexkit-cli" }   # lib before cli
    secrets: { CARGO_REGISTRY_TOKEN: ${{ secrets.CARGO_REGISTRY_TOKEN }} }
```

`.github/workflows/security-nightly.yml`:

```yaml
name: security-nightly
on:
  schedule: [{ cron: "0 6 * * *" }]
  workflow_dispatch: {}
jobs:
  sweep:
    uses: userFRM/kit-ci/.github/workflows/security-nightly.yml@<commit-sha>
    permissions: { issues: write, contents: read }
```

## Data-refresh nightly stays per-repo

Each kit's nightly data fetch differs (sources, CLI subcommands, SEC
User-Agent). It is NOT reusable — `templates/data-nightly.yml` is the shape to
copy and fill in. The commit-and-push tail is identical everywhere; only the
fetch steps change.

## Hygiene files

Copy from `templates/` into each kit root: `rust-toolchain.toml`,
`clippy.toml`, `deny.toml`, `audit.toml`. The `deny.toml`/`audit.toml`
advisory `ignore` lists are per-repo — keep the two in sync.
