# .github

*Community health files for the Commerce Layer GitHub organization.*

## Workflows

### Dependency update

Automates keeping dependencies up to date on **Node.js (pnpm)** projects. It runs `npm-check-updates`, commits the changes to a new branch, opens a tracking issue, and creates a pull request — all in one shot.

It can be triggered manually via `workflow_dispatch` or in response to a GitHub issue labeled `vulnerability`.

```yaml
name: Vulnerability update

on:
  issues:
    types: [opened, labeled, reopened]
  workflow_dispatch:

concurrency:
  group: vulnerability-update-${{ github.event_name }}-${{ github.event.issue.number || github.run_id }}
  cancel-in-progress: true

jobs:
  run:
    if: |
      github.event_name == 'workflow_dispatch' ||
      (
        github.event_name == 'issues' &&
        contains(github.event.issue.labels.*.name, 'vulnerability')
      )
    uses: commercelayer/.github/.github/workflows/dependencies-update-pnpm.yaml@main
    with:
      reviewers: "malessani,gciotola"
      use_milestone: false
      run_test: true
      run_build: true
      run_check: false
      issue_number: ${{ github.event_name == 'issues' && github.event.issue.number || '' }}
    secrets: inherit
```

#### Version inference

Toolchain versions are auto-detected from standard project files. An explicit workflow input always takes precedence; if omitted the following sources are checked in order.

**Node.js**

| Priority | Source |
|---|---|
| 1 | `node_version` workflow input |
| 2 | `.nvmrc` |
| 3 | `.node-version` |
| 4 | `package.json` → `volta.node` |
| 5 | `package.json` → `engines.node` (major extracted, e.g. `>=20` → `20.x`) |
| 6 | **default: `24.x`** |

**pnpm**

| Priority | Source |
|---|---|
| 1 | `pnpm_version` workflow input |
| 2 | `package.json` → `packageManager` (`pnpm@x.y.z`) — version passed implicitly; see note below |
| 3 | `package.json` → `volta.pnpm` |
| 4 | `package.json` → `engines.pnpm` |
| 5 | **default: `10.x`** |

#### Requirements & Known Limitations

- **`.ncurc.js` must not be present** in the target repository. `npm-check-updates` automatically loads this config file when found, which can override the flags passed by the workflow (e.g. `--packageFile`, `-u`, `-t semver`) and produce unexpected results. Remove or rename it before using this workflow.

- **Prefer `engines.pnpm` over `packageManager`** for specifying the pnpm version. When a `packageManager` field is present in `package.json`, `pnpm/action-setup` reads it automatically and conflicts with any explicit `version` input — the workflow works around this by skipping the version input entirely, which pins you to whatever exact version is in `packageManager`. Using `engines.pnpm` instead (e.g. `"10.x"`) gives you a flexible version range and avoids the conflict:

  ```json
  {
    "engines": {
      "node": ">=20",
      "pnpm": "10.x"
    }
  }
  ```

