# VERSION=2.21.1.0
# syntax=docker/dockerfile:1
#
# gyw-ansible — the container image that runs the Talos automation. It bundles
# ansible-core + talosctl (PINNED to the cluster's Talos version) + kubectl + the
# oracle.oci Ansible collection (OCI Vault I/O). It authenticates to OCI purely
# from environment variables injected by the misc-scripts/gyw-ansible.py
# wrapper (no ~/.oci/config file); the playbooks materialize the API-key PEM
# from OCI_USER_KEY_CONTENT to OCI_USER_KEY_FILE at start.
#
# YAML stdout is produced by ansible-core's built-in default callback with
# `callback_result_format = yaml` (ansible/ansible.cfg); the old
# community.general `yaml` callback was removed in community.general 12.0.
#
# Intended tag:  ghcr.io/initialgyw/gyw-ansible:2.21.1.0  (== the ansible-core version + revision)
# Multi-arch:    built for linux/arm64 (Apple-Silicon host) and linux/amd64.
#
# Build (single-arch, local Apple-Silicon):
#   docker build -t ghcr.io/initialgyw/gyw-ansible:2.21.1.0 \
#     -f gyw-ansible/Dockerfile .
#
# Build (multi-arch, push):
#   docker buildx build --platform linux/arm64,linux/amd64 \
#     -t ghcr.io/initialgyw/gyw-ansible:2.21.1.0 \
#     -f gyw-ansible/Dockerfile --push .
#
# The misc-scripts/gyw-ansible.py wrapper invokes this image with the homelab
# repo mounted at /workspace.

# --- Pinned tool versions ----------------------------------------------------
ARG TALOSCTL_VERSION=v1.13.5
ARG ANSIBLE_CORE_VERSION=2.21.1
ARG KUBECTL_VERSION=v1.36.2

# --- Stage 1: fetch the static CLIs for the target arch ----------------------
FROM --platform=$BUILDPLATFORM alpine:3.19 AS tools
ARG TALOSCTL_VERSION
ARG KUBECTL_VERSION
ARG TARGETARCH
RUN apk add --no-cache ca-certificates curl
WORKDIR /out
# talosctl — pinned to the cluster's Talos version (gen config templates are
# version-specific, so this pin MUST track group_vars/all.yaml talos_version).
RUN curl -fsSL -o talosctl \
      "https://github.com/siderolabs/talos/releases/download/${TALOSCTL_VERSION}/talosctl-linux-${TARGETARCH}" \
    && chmod 0755 talosctl

# kubectl
RUN curl -fsSL -o kubectl \
      "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/${TARGETARCH}/kubectl" \
    && chmod 0755 kubectl

# --- Stage 2: the runtime image ----------------------------------------------
FROM python:3.12-alpine AS runtime
ARG ANSIBLE_CORE_VERSION

LABEL org.opencontainers.image.source=https://github.com/initialgyw/pub-dockerfiles
LABEL org.opencontainers.image.title="gyw-ansible"
LABEL org.opencontainers.image.description="Built from /gyw-ansible/Dockerfile in pub-dockerfiles"
LABEL org.opencontainers.image.documentation=https://github.com/initialgyw/pub-dockerfiles/blob/main/gyw-ansible/README.md
LABEL org.opencontainers.image.version="2.21.1.0"

# ca-certificates is required for talosctl TLS and OCI SDK HTTPS.
# libffi + openssl are needed to build the cryptography wheel that the OCI
# Python SDK depends on; the build toolchain is added/removed in one layer.
RUN apk add --no-cache ca-certificates libffi openssl

# Python deps:
#   - ansible-core : the playbook engine
#   - pyyaml       : required by ansible-core
#   - oci          : the OCI Python SDK used by the oracle.oci collection for
#                    all OCI Vault reads/writes (auth via OCI_* env vars).
RUN apk add --no-cache --virtual .build-deps gcc musl-dev libffi-dev openssl-dev \
    && pip install --no-cache-dir \
      "ansible-core==${ANSIBLE_CORE_VERSION}" \
      "pyyaml>=6.0.0" \
      "oci" \
    && apk del .build-deps

# Ansible collections:
#   - oracle.oci : OCI Vault secret get/create/update + KMS/identity facts
RUN ansible-galaxy collection install oracle.oci

# The pinned CLIs from stage 1.
COPY --from=tools /out/talosctl /usr/local/bin/talosctl
COPY --from=tools /out/kubectl /usr/local/bin/kubectl


WORKDIR /workspace/ansible

# Entrypoint runs ansible-playbook by default.
ENTRYPOINT ["ansible-playbook"]
CMD ["--version"]
