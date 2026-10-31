# v0lt-systems/.github

Shared defaults for every repository of the V0LT Systems organization.

| Path | What it does |
|---|---|
| `profile/README.md` | The organization's public profile page. |
| `SECURITY.md` | Default security policy for repositories that have none. |
| `.github/workflows/runner-routing.yml` | Reusable workflow: picks the self-hosted `pc-gaming` runners, or `ubuntu-latest` when the PC is off. |
| `.github/workflows/dependabot-automerge.yml` | Reusable workflow, opt-in: auto-merges Dependabot patch and minor updates once required checks pass. |
| `workflow-templates/` | Templates offered under **Actions → New workflow** in every repository. |

## Runner routing

Call it from any repository of the organization:

```yaml
jobs:
  routing:
    uses: v0lt-systems/.github/.github/workflows/runner-routing.yml@main
  build:
    needs: routing
    runs-on: ${{ fromJSON(needs.routing.outputs.runner) }}
```

The switches are the organization variable `CI_RUNNER` (JSON for `runs-on`; set it to `"ubuntu-latest"` while the PC is off), the repository variable `RUNNER_FORCE` (`pc` or `github`) and `PC_RUNNER_VU`, the heartbeat the PC rewrites every minute. The header of the workflow explains the order.

commodity, famille, hillvalleyrealestate and lewine still carry their own copy (`./.github/workflows/runner-routing.yml`), identical to this one.

## Dependabot auto-merge (opt-in)

Only for repositories where merging to the default branch is not, by itself, a production deploy. famille's autonomy rules forbid it (the merge is the human gate there), and so do any repositories with the same rule.

```yaml
name: Dependabot auto-merge
on: pull_request
jobs:
  automerge:
    uses: v0lt-systems/.github/.github/workflows/dependabot-automerge.yml@main
    permissions:
      contents: write
      pull-requests: write
```

The repository needs **Allow auto-merge** on and a ruleset with required checks (`ci-ok`), or GitHub merges at once.
