# v0lt-systems/.github

Shared defaults for every repository of the V0LT Systems organization.

| Path | What it does |
|---|---|
| `profile/README.md` | The organization's public profile page. |
| `SECURITY.md` | Default security policy for repositories that have none. |
| `.github/workflows/runner-routing.yml` | Reusable workflow: picks the PC's `pc-gaming` runners, qcm1250 when the PC is full or off, `ubuntu-latest` only when both are off (asks Master of Puppets). |
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

Each run goes to the PC (`pc-gaming`) while it has a free runner, to qcm1250 when the PC is full or off, and to `ubuntu-latest` only when both are off (Vincent, 2026-11-01). Master of Puppets decides from GitHub's live runner list (`GET /api/ingest/runner-route`, masterofpuppets `docs/runners.md`). The routing job itself runs on any self-hosted Linux runner, or where the organization variable `CI_RUNNER` says: Master of Puppets sets it to qcm1250 while the PC is off and to `"ubuntu-latest"` only when both are. When Master of Puppets does not answer, the job falls back on `CI_RUNNER`, then the PC. The repository variable `RUNNER_FORCE` (`pc`, `qcm1250` or `github`) overrides everything.

Every repository of the organization calls this copy (commodity since its D50). `PC_RUNNER_VU`, the heartbeat the PC rewrites every minute, is only the last resort now.

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
