# setup-doover

Installs the Doover CLI and the toolchain its `doover app` commands need: `uv`,
Python, Node, and `doover-cli` itself.

## If you are setting up CI for a Doover app, you probably want the pipeline

This action installs tools and nothing else. The full pipeline — lint, test,
schema validation, in-container smoke test, build, push and release — is
maintained centrally and is what app repos should use:

```yaml
jobs:
  app:
    uses: getdoover/workflows/.github/workflows/app.yml@v1
    secrets: inherit
```

Reach for this action only when you are composing your own pipeline and want the
toolchain without the opinions:

```yaml
- uses: actions/checkout@v4
- uses: getdoover/setup-doover@v1
- run: doover app publish --app-name my_app --build
```

## No secrets

Inside GitHub Actions with `id-token: write`, the CLI detects the runner and
authenticates to Doover over the trusted-publisher OIDC flow. There is no API
token to configure.

```yaml
permissions:
  contents: read
  id-token: write
```

## Inputs

| Input | Default | |
|---|---|---|
| `python-version` | `3.11` | |
| `node-version` | `20` | Installed unconditionally — widgets are built with it, and deciding per app would mean branching before app discovery has run. |
| `cli-version` | `doover-cli` | Pin as `doover-cli==1.4.0`. Worth pinning in one shared workflow rather than per repo, so a bad release is one bump to undo. |

## Why this exists separately

The pipeline needs job-level features an action cannot declare — `container:` for
the test suite, `strategy.matrix` to fan out over the apps a repo contains, and
`fail-fast` so one broken app does not mask the others. So the pipeline is a
reusable workflow, and this is the setup block its jobs share.
