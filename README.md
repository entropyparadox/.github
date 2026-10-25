# entropyparadox/.github

Organization-wide reusable GitHub Actions workflows. No secrets live here.

## ep-pick: run on the company build pool, fall back to GitHub-hosted

`.github/workflows/ep-pick.yml` answers, per job, where it should run:

- `"ep-c<cpu>-m<GiB>"` (for example `ep-c2-m4` = 2 CPU / 4 GiB, `ep-c0p5-m8` = 0.5 CPU / 8 GiB): a runner of the
  EP build pool (k3s + Actions Runner Controller on company machines) sized from the job's measured history.
  CPU steps 0.5, 1, 2, 3, 4, 6, 8; memory steps 0.5, 1, 2, 3, 4, 6, 8, 12, 16, 24, 32 GiB.
- `"ubuntu-latest"`: the pool has no room for that size, is unhealthy, or its status is unknown.

Use it in a workflow (one line per job; index with the job's display name: its `name:`, or the job id):

```yaml
jobs:
  pick:
    uses: entropyparadox/.github/.github/workflows/ep-pick.yml@<commit sha>
    # Optional: a fixed size for some jobs, wins over measured history (snapped up to the steps above).
    # with:
    #   sizes: '{"build": "cpu=4/ram=16"}'
  build:
    needs: pick
    runs-on: ${{ fromJSON(needs.pick.outputs.map)['build'] || fromJSON(needs.pick.outputs.runs-on) }}
    steps: ...
```

Callers pinned to `6c703f18` (v2) keep working: they get the older labels `ep-linux-x64-s`, `ep-linux-x64`,
`ep-linux-x64-l`. Move to the newest commit of branch `ep-pick-v1` to get the finer sizes.

Pool runners are plain Ubuntu 24.04 containers (official `actions-runner` image): git, python3, jq,
curl, unzip and the docker CLI are present; Node, build tools and a Docker daemon are not, and
`sudo` does not work. Use setup actions (`actions/setup-node`, ...) for toolchains. Jobs that need
`sudo apt-get`, a Docker daemon, `services:` containers, macOS or production deploy credentials
should keep `runs-on: ubuntu-latest`.

Operations notes (pool host, status publisher, watchdog) are kept with the pool infrastructure (Task 60).
