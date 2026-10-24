# entropyparadox/.github

Organization-wide reusable GitHub Actions workflows. No secrets live here.

## ep-pick: run on the company build pool, fall back to GitHub-hosted

`.github/workflows/ep-pick.yml` answers where a job should run:

- `"ep-linux-x64"`: the EP build pool (k3s + Actions Runner Controller on company machines) has a free slot.
- `"ubuntu-latest"`: the pool is full, unhealthy, or its status is unknown.

Use it in a workflow:

```yaml
jobs:
  pick:
    uses: entropyparadox/.github/.github/workflows/ep-pick.yml@<commit sha>
  build:
    needs: pick
    runs-on: ${{ fromJSON(needs.pick.outputs.runs-on) }}
    steps: ...
```

Pool runners are plain Ubuntu 24.04 containers (official `actions-runner` image): git, python3, jq,
curl, unzip and the docker CLI are present; Node, build tools and a Docker daemon are not, and
`sudo` does not work. Use setup actions (`actions/setup-node`, ...) for toolchains. Jobs that need
`sudo apt-get`, a Docker daemon, `services:` containers, macOS or production deploy credentials
should keep `runs-on: ubuntu-latest`.

Operations notes (pool host, status publisher, watchdog) are kept with the pool infrastructure (Task 60).
