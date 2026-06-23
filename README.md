# sitehub-ci

Shared **reusable GitHub Actions workflows** for sitehub repos. One source of
truth for CI/CD — repos call these instead of copying job definitions.

## Project types

| Workflow | For | Does |
|---|---|---|
| [`rust-lib`](.github/workflows/rust-lib.yml) | publishable library workspace (e.g. `sitehub-common`) | fmt · deny · build · clippy · nextest → **publish to kellnr** on push to main |
| [`rust-api`](.github/workflows/rust-api.yml) | deployable HTTP service (e.g. `sitehub-iam`, `sitehub-gateway`) | fmt · deny · build · clippy · nextest → **deploy to Fly** + post-deploy health gate |

Both: `Swatinem/rust-cache` (surrealdb compiled once, cached), `RUSTFLAGS=-D warnings`,
toolchain pinned via input (default 1.96), `--locked`.

## Usage

`rust-lib` (`.github/workflows/ci.yml` in the caller):
```yaml
name: CI
on: { pull_request: {}, push: { branches: [main] } }
jobs:
  ci:
    uses: sitehub-bg/sitehub-ci/.github/workflows/rust-lib.yml@v1
    with:
      publish-crates: "crate-a crate-b"   # dependency order; omit to skip publish
    secrets:
      KELLNR_TOKEN: ${{ secrets.KELLNR_TOKEN }}
```

`rust-api`:
```yaml
name: CI/CD
on: { pull_request: {}, push: { branches: [main] } }
jobs:
  cicd:
    uses: sitehub-bg/sitehub-ci/.github/workflows/rust-api.yml@v1
    with:
      app: sitehub-iam
      health-url: https://iam.sitehub.bg/api/ready
    secrets:
      FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}
```

## Notes

- Pin callers to a tag (`@v1`) for stability; bump deliberately. `@main` floats.
- The caller repo supplies its local config files (`deny.toml`, `rustfmt.toml`,
  `clippy.toml`, `rust-toolchain.toml`, `.cargo/config.toml`) — tooling reads
  those locally, so they cannot be centralized here. Keep them aligned via the
  service template + dependabot.
