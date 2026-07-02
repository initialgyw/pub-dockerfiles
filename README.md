# pub-dockerfiles

## opentofu

### Building and tagging image

```bash
# Build for multiple platforms
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t ghcr.io/initialgyw/opentofu:1.12.0-alpine-3.23.4 \
  -t ghcr.io/initialgyw/opentofu:1.12.0-alpine \
  -t ghcr.io/initialgyw/opentofu:1.12-alpine \
  -t ghcr.io/initialgyw/opentofu:alpine \
  -f opentofu/Dockerfile \
  --push opentofu/
```

## ansible-talos

```zsh
ghcr.io/initialgyw/ansible-talos:1.13.5

# Build for multiple platforms
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t ghcr.io/initialgyw/ansible-talos:1.13.5-alpine \
  -f ansible-talos/Dockerfile \
  --push ansible-talos/
```

Override any pin at build time with `--build-arg`, e.g.
`--build-arg TALOSCTL_VERSION=v1.13.6`.

```bash
# Single-arch, local Apple-Silicon (arm64) host:
docker build -t ghcr.io/initialgyw/ansible-talos:1.13.5 \
  -f ansible-talos/Dockerfile .

# Multi-arch (arm64 + amd64), push to GHCR:
docker buildx build --platform linux/arm64,linux/amd64 \
  -t ghcr.io/initialgyw/ansible-talos:1.13.5 \
  -f ansible-talos/Dockerfile --push .
```
