# VERSION=1.13.6.1
# syntax=docker/dockerfile:1
#
# ansible-talos — the credential-injecting container image that runs the Talos
# automation. It bundles ansible-core + talosctl (PINNED to the cluster's Talos
# version).
#
# Intended tag:  ghcr.io/initialgyw/ansible-talos:1.13.6.1  (== the Talos version)
# Multi-arch:    built for linux/arm64 (Apple-Silicon host) and linux/amd64.
#
# Build (single-arch, local Apple-Silicon):
#   docker build -t ghcr.io/initialgyw/ansible-talos:1.13.6.1 \
#     -f ansible-talos/Dockerfile .
#
# Build (multi-arch, push):
#   docker buildx build --platform linux/arm64,linux/amd64 \
#     -t ghcr.io/initialgyw/ansible-talos:1.13.6.1 \
#     -f ansible-talos/Dockerfile --push .
#
# The misc-scripts/ansible-talos.py wrapper invokes this image with the homelab
# repo mounted at /workspace.

# --- Pinned tool versions ----------------------------------------------------
ARG TALOSCTL_VERSION=v1.13.5
ARG ANSIBLE_CORE_VERSION=2.21.1

# --- Stage 1: fetch the static CLIs for the target arch ----------------------
FROM --platform=$BUILDPLATFORM alpine:3.19 AS tools
ARG TALOSCTL_VERSION
ARG TARGETARCH
RUN apk add --no-cache ca-certificates curl
WORKDIR /out
# talosctl — pinned to the cluster's Talos version (gen config templates are
# version-specific, so this pin MUST track group_vars/all.yaml talos_version).
RUN curl -fsSL -o talosctl \
      "https://github.com/siderolabs/talos/releases/download/${TALOSCTL_VERSION}/talosctl-linux-${TARGETARCH}" \
    && chmod 0755 talosctl

# --- Stage 2: the runtime image ----------------------------------------------
FROM python:3.12-alpine AS runtime
ARG ANSIBLE_CORE_VERSION

LABEL org.opencontainers.image.source=https://github.com/initialgyw/pub-dockerfiles
LABEL org.opencontainers.image.title="ansible-talos"
LABEL org.opencontainers.image.description="Built from /ansible-talos/Dockerfile in pub-dockerfiles"
LABEL org.opencontainers.image.documentation=https://github.com/initialgyw/pub-dockerfiles/blob/main/ansible-talos/README.md
LABEL org.opencontainers.image.version="1.13.6.1"

# ca-certificates is required for talosctl TLS and OCI SDK HTTPS.
# Added libffi for python cryptography wheel support (if needed by ansible/oci)
RUN apk add --no-cache ca-certificates libffi

# Python deps: ansible-core (the playbook engine) + pyyaml (ansible-core needs it).
RUN pip install --no-cache-dir \
      "ansible-core==${ANSIBLE_CORE_VERSION}" \
      "pyyaml>=6.0.0"

# Install community.general for the yaml stdout callback plugin
RUN ansible-galaxy collection install community.general

# The pinned CLIs from stage 1.
COPY --from=tools /out/talosctl /usr/local/bin/talosctl


WORKDIR /workspace/ansible

# Entrypoint runs ansible-playbook by default.
ENTRYPOINT ["ansible-playbook"]
CMD ["--version"]
