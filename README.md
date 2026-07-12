# renovate-config

Centralized Renovate configuration preset

## Presets

### `default.json`

Extend with `"github>miikkak/renovate-config"`. Provides:

- Automerge for GitHub Actions minor/patch updates (major stays manual)
- Automerge for pre-commit hook minor/patch updates (major stays manual)
- Automerge for npm devDependency minor/patch updates (major stays manual)
- `platformAutomerge` enabled

### `release-none.json`

Extend with `"github>miikkak/renovate-config:release-none"` in repos that
enforce a mandatory `release:` label (`.github/label-check.yml` with
`required: true`). Adds the `release:none` label to npm devDependency update
PRs, since those only touch release tooling (semantic-release toolchain) and
must not trigger a deployment. This lets the release-label check pass so
automerge can proceed.
