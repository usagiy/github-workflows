# github-workflows

Reusable GitHub Actions workflows shared across application repos.

## `docker-build.yml` — build & push an image to GHCR

Drop this into any app repo as `.github/workflows/build.yml`:

```yaml
name: Build
on:
  push:
    branches: [main]
    tags: ["v*"]
  pull_request:

jobs:
  build:
    uses: usagiy/github-workflows/.github/workflows/docker-build.yml@main
    permissions:
      contents: read
      packages: write
    # All inputs are optional:
    # with:
    #   dockerfile: ./Dockerfile
    #   context: .
    #   test-command: "npm ci && npm test"
    #   push-latest: true
```

### Behaviour

| Trigger | Result |
|---|---|
| `pull_request` | build only (validates the Dockerfile), **no push** |
| push to `main` | build **and** push `ghcr.io/usagiy/<repo>:sha-<short>` (+ `latest`) |
| push tag `vX.Y.Z` | additionally push `ghcr.io/usagiy/<repo>:X.Y.Z` |

- Auth uses the built-in `GITHUB_TOKEN` (`packages: write`) — no extra secrets.
- Images are **amd64 only** (the cluster runs on x86 `cx23` nodes).
- `latest` is a convenience tag; **deployments always pin the immutable
  `sha-`/`vX.Y.Z` tag** in `cluster-config`.

### Deploying what you built

This workflow only produces images. To run one, promote it manually in the
`cluster-config` repo: set `images[].newTag` to the `sha-...` you want and push.
ArgoCD syncs it. (Build is automated; the release decision is yours.)
