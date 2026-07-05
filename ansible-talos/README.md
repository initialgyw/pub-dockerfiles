# ansible-talos

The container image that runs the Talos automation. It bundles `ansible-core` +
`talosctl` (pinned to the cluster's Talos version) + `kubectl` + the `oracle.oci`
Ansible collection (OCI Vault I/O).

- **Image:** `ghcr.io/initialgyw/ansible-talos`
- **Version:** `1.13.6.3` (equals the cluster's Talos version; see the
  `# VERSION=` header in the [`Dockerfile`](./Dockerfile))
- **Source:** [`ansible-talos/Dockerfile`](./Dockerfile)

## Authentication

The image authenticates to OCI **purely from environment variables** (no
`~/.oci/config` file, no mounted key). The `misc-scripts/ansible-talos.py`
wrapper parses `script_outputs/initialize-oci-summary.json` and injects:

- OCI SDK auth (read by `oracle.oci` / the OCI Python SDK): `OCI_TENANCY`,
  `OCI_USER_ID`, `OCI_USER_FINGERPRINT`, `OCI_REGION`, `OCI_USER_KEY_FILE`,
  `OCI_USER_KEY_PASS_PHRASE`.
- Playbook-managed: `OCI_USER_KEY_CONTENT` (the PEM, written to
  `OCI_USER_KEY_FILE` at 0400 by an early task), `OCI_VAULT_ID`,
  `OCI_COMPARTMENT_ID`, `OCI_MEK_KEY_ID`.

## Bundled Ansible collections

| Collection | Purpose |
|---|---|
| `oracle.oci` | OCI Vault secret get/create/update + KMS/identity facts |

YAML stdout is produced by ansible-core's built-in `default` callback with
`callback_result_format = yaml` (`ansible/ansible.cfg`) — the old
`community.general.yaml` callback was removed in community.general 12.0.


## Tags

The CI workflow ([`.github/workflows/docker.yml`](../.github/workflows/docker.yml))
publishes a single multi-arch tag matching the `# VERSION=` header in the
Dockerfile:

| Tag | Platforms |
|---|---|
| `ghcr.io/initialgyw/ansible-talos:1.13.6.3` | `linux/amd64`, `linux/arm64` |

## Pinned Tool Versions

The following versions are pinned via build args in the
[`Dockerfile`](./Dockerfile). `talosctl` **must** track the cluster's Talos
version (`group_vars/all.yaml` `talos_version`), because the `gen config`
templates are version-specific.

| Build arg | Default |
|---|---|
| `TALOSCTL_VERSION` | `v1.13.5` |
| `ANSIBLE_CORE_VERSION` | `2.21.1` |
| `KUBECTL_VERSION` | `v1.36.2` |

## Usage

The entrypoint is `ansible-playbook` (default `CMD` is `--version`), and the
working directory is `/workspace/ansible`. The `misc-scripts/ansible-talos.py` wrapper invokes this image with the homelab repo mounted at `/workspace`.

Run a playbook by mounting the homelab repo at `/workspace`:

```sh
docker run --rm -it \
  -v ${PWD}:/workspace \
  ghcr.io/initialgyw/ansible-talos:1.13.6.3 \
  site.yaml
```

## Building Locally

Override any pinned tool at build time with `--build-arg`, e.g.
`--build-arg TALOSCTL_VERSION=v1.13.6`.

```bash
# Single-arch, local Apple-Silicon (arm64) host:
docker build -t ghcr.io/initialgyw/ansible-talos:1.13.6.3 \
  -f ansible-talos/Dockerfile .

# Multi-arch (arm64 + amd64), push to GHCR:
docker buildx build --platform linux/arm64,linux/amd64 \
<<<<<<< HEAD
  -t ghcr.io/initialgyw/ansible-talos:1.13.6.3 \
=======
  -t ghcr.io/initialgyw/ansible-talos:1.13.6.2 \
>>>>>>> main
  -f ansible-talos/Dockerfile --push .
```
