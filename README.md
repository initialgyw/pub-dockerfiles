# pub-dockerfiles

## Building and tagging image

```
# Build for multiple platforms
docker build build \
  --platform linux/amd64,linux/arm64 \
  -t ghcr.io/initialgyw/opentofu:1.12.0-alpine-3.23.4 \
  -t ghcr.io/initialgyw/opentofu:1.12.0-alpine \
  -t ghcr.io/initialgyw/opentofu:1.12-alpine \
  -t ghcr.io/initialgyw/opentofu:alpine \
  --push .
```
