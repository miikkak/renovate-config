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
  `release:minor`/`release:patch` (major stays manual, unlabeled) — except
  JUnit and Mockito (`org.junit*`/`org.mockito*`), which are always
  test-scoped and always automerge labeled `release:none`, major included
- Automerge for Go module minor/patch updates, labeled
  `release:minor`/`release:patch` (major stays manual, unlabeled)
- Automerge for Dockerfile/Containerfile base image digest/patch/minor
  updates (native `dockerfile` manager, i.e. `FROM` lines not tracked by a
  custom regex ARG manager), labeled `release:patch`/`release:minor` (major
  stays manual, unlabeled)
- Automerge for PEP 621 (`pyproject.toml`) dependency minor/patch updates,
  labeled `release:minor`/`release:patch` (major stays manual, unlabeled) —
  except `dependency-groups.dev` deps (pytest, ruff, etc.), which always
  automerge labeled `release:none`, major included
- Never bumps the `scop/pre-commit-shfmt` hook pin; it must equal the shfmt
  bundled in the Super-Linter image. `super-linter.yml` in `miikkak/workflows`
  syncs it on Renovate PRs that move the workflow digest and fails the check on
  drift everywhere else
- Ignores commits by `github-actions[bot]` when deciding whether a Renovate PR
  was edited (`gitIgnoredAuthors`). The shfmt pin sync in `super-linter.yml`
  pushes a commit to Renovate's digest PRs; without this, Renovate would treat
  the PR as modified and stop rebasing it. Do not add a human's email here:
  Renovate would then rebase over their manual edits.
- `platformAutomerge` enabled
- `rebaseWhen: conflicted`: Renovate rebases a PR only when it conflicts with
  the base branch, not every time another PR merges. See below.

### Rebasing and strict required status checks

Renovate's default (`rebaseWhen: auto`) rebases every behind PR whenever
automerge is configured, which this preset does for almost every rule. Where a
merge does not require the branch to be up to date, that only multiplies CI and
review runs: each merge makes every other open PR rebase and re-run.

So this preset uses `conflicted`. The catch, from Renovate's own docs: with
`conflicted`, a repo whose ruleset **requires branches to be up to date** (the
`strict` setting in `miikkak/workflows` `governance/`) would leave behind PRs
stuck, because nothing would rebase them and GitHub would refuse to merge them.

Such a repo must override it in its own `renovate.json`:

```json
{
  "extends": ["github>miikkak/renovate-config"],
  "rebaseWhen": "behind-base-branch"
}
```

Keep that override exactly as long as the repo is listed with `strict: true` in
`governance/repos.yaml`, and remove it in the same change that relaxes strict.
Renovate's documentation only describes detecting this from branch protection
and does not mention rulesets, which is what these repos use, so do not rely on
`auto` to work it out.

### Known gap: other test-only Gradle/Maven dependencies

Some repos declare non-framework packages (e.g. `gson`, `velocity-api`) as
`testImplementation`-only, but Renovate's gradle manager can't expose that
scoping as a matchable `depType` (unlike npm/pep621), and the package names
themselves aren't reliably test-only across every repo — `gson` could be a
real runtime dependency elsewhere. These still get labeled
`release:patch`/`release:minor` by the general Gradle/Maven rule even when,
in a given repo, they're test-only. Fixing this precisely requires a
per-repo `matchPackageNames` override in that repo's own `renovate.json`,
not a change here.

### `release-none.json`

Extend with `"github>miikkak/renovate-config:release-none"` in repos that
enforce a mandatory `release:` label (`.github/label-check.yml` with
`required: true`). Adds the `release:none` label to npm devDependency update
PRs, since those only touch release tooling (semantic-release toolchain), and
to GitHub Actions update PRs (overriding the `release:patch`/`release:minor`
labels from `default.json`), since those only touch workflow files. Neither
may trigger a deployment. This lets the release-label check pass so
automerge can proceed.
