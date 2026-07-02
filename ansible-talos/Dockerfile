# VERSION=1.13.5
# syntax=docker/dockerfile:1
#
# ansible-talos — the credential-injecting container image that runs the Talos
# automation. It bundles ansible-core + talosctl (PINNED to the cluster's Talos
# version) + the gywadmin-oci package, which provides the OCI
# Vault helper CLIs (manage-vault / initialize-oci / update-github-secrets) and
# pulls in the OCI Python SDK, so the playbooks can read/write cluster state
# (secrets.yaml, talosconfig, kubeconfig) to OCI Vault.
#
# The OCI helpers used to be COPY'd out of the gywadmin-homelab repo (which
# forced the build context to be that repo's root). They now live in the
# standalone public package github.com/initialgyw/gywadmin-oci and are pip
# installed at build time (pinned via GYWADMIN_OCI_VERSION), so this image no
# longer depends on the homelab repo — nothing is COPY'd from the build context,
# so the context can be anything (e.g. the pub-dockerfiles root).
#
# Intended tag:  ghcr.io/initialgyw/ansible-talos:1.13.5  (== the Talos version)
# Multi-arch:    built for linux/arm64 (Apple-Silicon host) and linux/amd64.
#
# Build (single-arch, local Apple-Silicon):
#   docker build -t ghcr.io/initialgyw/ansible-talos:1.13.5 \
#     -f ansible-talos/Dockerfile .
#
# Build (multi-arch, push):
#   docker buildx build --platform linux/arm64,linux/amd64 \
#     -t ghcr.io/initialgyw/ansible-talos:1.13.5 \
#     -f ansible-talos/Dockerfile --push .
#
# The misc-scripts/ansible-talos.py wrapper invokes this image with the homelab
# repo mounted at /workspace; the playbooks call the Vault helper via the
# `manage-vault` console script on PATH (see group_vars: manage_vault_cmd).

# --- Pinned tool versions ----------------------------------------------------
ARG TALOSCTL_VERSION=v1.13.5
ARG ANSIBLE_CORE_VERSION=2.21.1
# gywadmin-oci: the OCI Vault helper CLIs (manage-vault / initialize-oci /
# update-github-secrets) + the OCI SDK are pulled in via this pip-installable
# package (it pins oci==2.181.0 + cryptography). Bump this to the git tag/branch
# you want baked into the image.
ARG GYWADMIN_OCI_VERSION=0.1.1

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
ARG GYWADMIN_OCI_VERSION

LABEL org.opencontainers.image.source=https://github.com/initialgyw/pub-dockerfiles
LABEL org.opencontainers.image.title="ansible-talos"
LABEL org.opencontainers.image.description="Built from /ansible-talos/Dockerfile in pub-dockerfiles"
LABEL org.opencontainers.image.documentation=https://github.com/initialgyw/pub-dockerfiles/blob/main/ansible-talos/README.md
LABEL org.opencontainers.image.version="1.13.5"

# ca-certificates is required for talosctl TLS and OCI SDK HTTPS.
# Added libffi for python cryptography wheel support (if needed by ansible/oci)
RUN apk add --no-cache ca-certificates libffi

# Python deps: ansible-core (the playbook engine) + pyyaml (ansible-core needs
# it) + the gywadmin-oci package, which provides the manage-vault / initialize-oci
# / update-github-secrets console scripts and pulls in its own pinned deps
# (oci==2.181.0, cryptography) transitively. oci/cryptography/pyyaml all ship
# prebuilt musllinux cp312 wheels, so no compiler toolchain is needed — only a
# transient `git` for pip's git+https clone of the gywadmin-oci package.
RUN apk add --no-cache --virtual .fetch-deps git \
    && pip install --no-cache-dir \
      "ansible-core==${ANSIBLE_CORE_VERSION}" \
      "pyyaml>=6.0.0" \
      "gywadmin-oci @ git+https://github.com/initialgyw/gywadmin-oci.git@${GYWADMIN_OCI_VERSION}" \
    && apk del .fetch-deps

# The pinned CLIs from stage 1.
COPY --from=tools /out/talosctl /usr/local/bin/talosctl

# The OCI Vault helper CLIs come from the pip-installed gywadmin-oci package
# (installed above): manage-vault / initialize-oci / update-github-secrets are on
# PATH at /usr/local/bin. The playbooks invoke `manage-vault` directly (see
# group_vars: manage_vault_cmd) — no in-image COPY of the helper is needed.

WORKDIR /workspace/ansible

# Entrypoint runs ansible-playbook by default. Override the entrypoint to run
# the Vault helper directly, e.g.:
#   docker run --rm --entrypoint manage-vault <image> list-secrets
ENTRYPOINT ["ansible-playbook"]
CMD ["--version"]
