FROM ghcr.io/opentofu/opentofu:1.12.0-minimal AS tofu

FROM alpine:3.23.4
LABEL org.opencontainers.image.source https://github.com/initialgyw/pub-dockerfiles/opentofu

COPY --from=tofu /usr/local/bin/tofu /usr/local/bin/tofu

WORKDIR /workspace

ENTRYPOINT ["tofu"]
