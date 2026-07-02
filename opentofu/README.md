# opentofu

An [Alpine](https://alpinelinux.org/)-based image that bundles the
[OpenTofu](https://opentofu.org/) `tofu` binary, copied from the official
`ghcr.io/opentofu/opentofu` minimal image, with `tofu` as the entrypoint.

- **Image:** `ghcr.io/initialgyw/opentofu`
- **Version:** `1.12.0` (the OpenTofu release; see the `# VERSION=` header in the
  [`Dockerfile`](./Dockerfile))
- **Source:** [`opentofu/Dockerfile`](./Dockerfile)

## Tags

The CI workflow ([`.github/workflows/docker.yml`](../.github/workflows/docker.yml))
publishes a single multi-arch tag matching the `# VERSION=` header in the
Dockerfile:

| Tag | Platforms |
|---|---|
| `ghcr.io/initialgyw/opentofu:1.12.0` | `linux/amd64`, `linux/arm64` |

## Usage

The image sets `WORKDIR /workspace` and uses `tofu` as its entrypoint. Mount your
Terraform/OpenTofu configuration into `/workspace` and pass `tofu` subcommands:

```sh
docker run --rm -it \
  -v ${PWD}:/workspace \
  ghcr.io/initialgyw/opentofu:1.12.0 \
  init
```

Replace `init` with any `tofu` subcommand (for example `plan`, `apply`,
`validate`).

## Building Locally

```bash
# Build for multiple platforms and push to GHCR
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t ghcr.io/initialgyw/opentofu:1.12.0 \
  -f opentofu/Dockerfile \
  --push opentofu/
```
