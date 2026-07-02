# syntax=docker/dockerfile:1
#
# ansible-talos — the credential-injecting container image that runs the Talos
# automation. It bundles ansible-core + talosctl (PINNED to the cluster's Talos
# version) + kubectl + helm + the OCI Python SDK, and copies the OCI Vault helper
# CLI (misc-scripts/oci/manage-vault.py + _oci_common.py) so the playbooks can
# read/write cluster state (secrets.yaml, talosconfig, kubeconfig) to OCI Vault.
#
# Intended tag:  ghcr.io/initialgyw/ansible-talos:1.13.5  (== the Talos version)
# Multi-arch:    built for linux/arm64 (Apple-Silicon host) and linux/amd64.
#
# Build (single-arch, local Apple-Silicon):
#   docker build -t ghcr.io/initialgyw/ansible-talos:1.13.5 \
#     -f Dockerfile .
#
# Build (multi-arch, push):
#   docker buildx build --platform linux/arm64,linux/amd64 \
#     -t ghcr.io/initialgyw/ansible-talos:1.13.5 \
#     -f Dockerfile --push .
#
# NOTE: the build context MUST be the repo root (so the COPY lines below can
# reach misc-scripts/oci/). The misc-scripts/ansible-talos.py wrapper always
# invokes this image with the repo mounted at /workspace, so the COPY is a
# convenience fallback (an in-container copy of the helper independent of the
# mount); the playbooks call the helper via the mounted /workspace path by
# default (see group_vars: manage_vault_cmd).

# --- Pinned tool versions ----------------------------------------------------
ARG TALOSCTL_VERSION=v1.13.5
ARG KUBECTL_VERSION=v1.36.2
ARG HELM_VERSION=v3.21.2
ARG ANSIBLE_CORE_VERSION=2.21.1
ARG OCI_SDK_VERSION=2.181.0

# --- Stage 1: fetch the static CLIs for the target arch ----------------------
FROM --platform=$BUILDPLATFORM alpine:3.19 AS tools
ARG TALOSCTL_VERSION
ARG KUBECTL_VERSION
ARG HELM_VERSION
ARG TARGETARCH
RUN apk add --no-cache ca-certificates curl
WORKDIR /out
# talosctl — pinned to the cluster's Talos version (gen config templates are
# version-specific, so this pin MUST track group_vars/all.yaml talos_version).
RUN curl -fsSL -o talosctl \
      "https://github.com/siderolabs/talos/releases/download/${TALOSCTL_VERSION}/talosctl-linux-${TARGETARCH}" \
    && chmod 0755 talosctl
# kubectl.
RUN curl -fsSL -o kubectl \
      "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/${TARGETARCH}/kubectl" \
    && chmod 0755 kubectl
# helm (v3) — extract the single binary out of the release tarball.
RUN curl -fsSL -o helm.tar.gz \
      "https://get.helm.sh/helm-${HELM_VERSION}-linux-${TARGETARCH}.tar.gz" \
    && tar -xzf helm.tar.gz \
    && mv "linux-${TARGETARCH}/helm" helm \
    && chmod 0755 helm \
    && rm -rf helm.tar.gz "linux-${TARGETARCH}"

# --- Stage 2: the runtime image ----------------------------------------------
FROM python:3.12-alpine AS runtime
ARG ANSIBLE_CORE_VERSION
ARG OCI_SDK_VERSION

# ca-certificates is required for talosctl/kubectl/helm TLS and OCI SDK HTTPS.
# Added libffi for python cryptography wheel support (if needed by ansible/oci)
RUN apk add --no-cache ca-certificates libffi

# Python deps: ansible-core (the playbook engine) + the OCI SDK and its friends
# that manage-vault.py / _oci_common.py import (oci, cryptography, pyyaml).
# We install build dependencies temporarily to compile any native extensions
# (like cryptography/pyyaml) if pre-built musllinux wheels aren't available.
RUN apk add --no-cache --virtual .build-deps \
        gcc \
        musl-dev \
        libffi-dev \
        openssl-dev \
        cargo \
    && pip install --no-cache-dir \
      "ansible-core==${ANSIBLE_CORE_VERSION}" \
      "oci==${OCI_SDK_VERSION}" \
      "cryptography>=42.0.0" \
      "pyyaml>=6.0.0" \
    && apk del .build-deps

# The pinned CLIs from stage 1.
COPY --from=tools /out/talosctl /usr/local/bin/talosctl
COPY --from=tools /out/kubectl /usr/local/bin/kubectl
COPY --from=tools /out/helm /usr/local/bin/helm

# The OCI Vault helper CLI + its shared library. Copied to a stable in-container
# path so the image can run manage-vault.py even without the repo mount; the
# playbooks default to the mounted /workspace copy (manage_vault_cmd).
COPY misc-scripts/oci/manage-vault.py /opt/oci/manage-vault.py
COPY misc-scripts/oci/_oci_common.py /opt/oci/_oci_common.py
RUN chmod 0755 /opt/oci/manage-vault.py

WORKDIR /workspace/ansible

# Entrypoint runs ansible-playbook by default. Override the entrypoint to run
# the Vault helper directly, e.g.:
#   docker run --rm --entrypoint python3 <image> /opt/oci/manage-vault.py list-secrets
ENTRYPOINT ["ansible-playbook"]
CMD ["--version"]
