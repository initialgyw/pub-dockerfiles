# VERSION=1.12.0
FROM ghcr.io/opentofu/opentofu:1.12.0-minimal AS tofu

FROM alpine:3.23.4
LABEL org.opencontainers.image.source=https://github.com/initialgyw/pub-dockerfiles
LABEL org.opencontainers.image.title="OpenTofu"
LABEL org.opencontainers.image.description="Built from /opentofu/Dockerfile in pub-dockerfiles"
LABEL org.opencontainers.image.documentation=https://github.com/initialgyw/pub-dockerfiles/blob/main/opentofu/README.md
LABEL org.opencontainers.image.version="1.12.0"

COPY --from=tofu /usr/local/bin/tofu /usr/local/bin/tofu

WORKDIR /workspace

ENTRYPOINT ["tofu"]
