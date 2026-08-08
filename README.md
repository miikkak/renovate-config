# renovate-config

Centralized Renovate configuration preset

## Presets

### `default.json`

Extend with `"github>miikkak/renovate-config"`. Provides:

- Automerge for GitHub Actions minor/patch updates (major stays manual)
- Automerge for pre-commit hook minor/patch updates (major stays manual)
- Automerge for npm devDependency minor/patch updates (major stays manual)
- Automerge for pip requirements minor/patch updates (major stays manual)
- Automerge for Gradle wrapper minor/patch updates, labeled `release:none`
  (it's build tooling, not a runtime dependency, so it never changes the
  compiled jar); major stays manual but is still labeled `release:none`
- Automerge for Gradle/Maven dependency minor/patch updates, labeled
  `release:minor`/`release:patch` (major stays manual, unlabeled)
- Automerge for Go module minor/patch updates, labeled
  `release:minor`/`release:patch` (major stays manual, unlabeled)
- Automerge for Dockerfile/Containerfile base image digest/patch/minor
  updates (native `dockerfile` manager, i.e. `FROM` lines not tracked by a
  custom regex ARG manager), labeled `release:patch`/`release:minor` (major
  stays manual, unlabeled)
- Automerge for PEP 621 (`pyproject.toml`) dependency minor/patch updates,
  labeled `release:minor`/`release:patch` (major stays manual, unlabeled)
- `platformAutomerge` enabled

### `release-none.json`

Extend with `"github>miikkak/renovate-config:release-none"` in repos that
enforce a mandatory `release:` label (`.github/label-check.yml` with
`required: true`). Adds the `release:none` label to npm devDependency update
PRs, since those only touch release tooling (semantic-release toolchain) and
must not trigger a deployment. This lets the release-label check pass so
automerge can proceed.
