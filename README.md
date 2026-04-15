# skynet-ci

Reusable GitHub Actions workflows for Skynet components. One source of truth for buildx config, Nexus auth, and cache routing.

## Usage

Consumer repo `sanscfs/skynet-<component>/.github/workflows/build.yml`:

```yaml
name: build
on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  build:
    uses: sanscfs/skynet-ci/.github/workflows/build.yml@main
    with:
      component: skynet-<component>
    secrets: inherit
```

The workflow:
- Uses the shared persistent BuildKit daemon (`buildkitd.github-runner.svc:1234`) — cache survives across all Skynet builds, not just one repo.
- Pushes multi-arch images to both `nexus.nexus.svc:8083/<component>` (in-cluster) and `ghcr.io/sanscfs/<component>` (external).
- Registers buildcache at `nexus.nexus.svc:8083/<component>:buildcache` with `mode=max`.

## Required repo secret

Each consumer repo needs `NEXUS_PASSWORD` from Vault `secret/nexus/admin`:

```bash
kubectl -n nexus get secret nexus-admin -o jsonpath='{.data.password}' | base64 -d \
  | gh secret set NEXUS_PASSWORD -R sanscfs/skynet-<component>
```

(Once `NEXUS_PASSWORD` is promoted to an org-level secret, this step goes away.)

## Inputs

| Input | Default | Purpose |
|-------|---------|---------|
| `component` | required | image name (also consumer-repo dir name) |
| `context` | `.` | build context |
| `dockerfile` | `Dockerfile` | relative to context |
| `platforms` | `linux/amd64,linux/arm64` | buildx platforms |
| `build-args` | empty | extra `--build-arg` lines |
| `extra-tags` | empty | additional tag lines (fully-qualified) |
| `tag-latest` | `true` | also push rolling tag |
| `rolling-tag` | `latest` | rolling tag value (`latest`, `py3.12`, etc.) |

See `sanscfs/infra/docs/github-runner-caching.md` for the full caching architecture.
