# v0lt-systems/.github

Shared defaults for every repository of the V0LT Systems organization.

| Path | What it does |
|---|---|
| `profile/README.md` | The organization's public profile page. |
| `SECURITY.md` | Default security policy for repositories that have none. |
| `.github/workflows/runner-routing.yml` | Reusable workflow: picks the PC's `pc-gaming` runners, qcm1250 when the PC is full or off, never GitHub-hosted: with both off the run queues for the first self-hosted Linux runner back (asks Master of Puppets). |
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

Each run goes to the PC (`pc-gaming`) while it has a free runner, to qcm1250 when the PC is full or off, and never to a GitHub-hosted runner: with both off it queues on `["self-hosted","Linux"]` for the first self-hosted runner back (Vincent, 2026-11-23: "I want to use my self hosted runners"). Master of Puppets decides from GitHub's live runner list (`GET /api/ingest/runner-route`, masterofpuppets `docs/runners.md`); Mac answers are used only by callers that pass `mac_ok`. The routing job itself runs on any self-hosted Linux runner, or where the organization variable `CI_RUNNER` says: Master of Puppets sets it to qcm1250 while the PC is off and to `["self-hosted","Linux"]` when both are. When Master of Puppets does not answer, the job follows `CI_RUNNER` only if it names a self-hosted runner, then the PC if the routing job itself ran self-hosted, else any self-hosted Linux runner. The repository variable `RUNNER_FORCE` (`pc` or `qcm1250`) overrides everything; `github` is no longer honoured. Master of Puppets opens an incident for any job that still asks for a GitHub-hosted runner.

Every repository of the organization calls this copy (commodity since its D50).

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
