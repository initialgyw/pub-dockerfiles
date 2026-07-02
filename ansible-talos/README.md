# ansible-talos

The credential-injecting container image that runs the Talos automation. It
bundles `ansible-core` + `talosctl` (pinned to the cluster's Talos version) + the
[`gywadmin-oci`](https://github.com/initialgyw/gywadmin-oci) package, which
provides the OCI Vault helper CLIs (`manage-vault` / `initialize-oci` /
`update-github-secrets`) and pulls in the OCI Python SDK, so the playbooks can
read/write cluster state (`secrets.yaml`, `talosconfig`, `kubeconfig`) to OCI
Vault.

- **Image:** `ghcr.io/initialgyw/ansible-talos`
- **Version:** `1.13.5` (equals the cluster's Talos version; see the
  `# VERSION=` header in the [`Dockerfile`](./Dockerfile))
- **Source:** [`ansible-talos/Dockerfile`](./Dockerfile)

The `gywadmin-oci` helpers are `pip install`ed at build time (pinned via the
`GYWADMIN_OCI_VERSION` build arg), so this image no longer depends on the
homelab repo — nothing is `COPY`'d from the build context, and the context can
be anything (for example the `pub-dockerfiles` root).

## Tags

The CI workflow ([`.github/workflows/docker.yml`](../.github/workflows/docker.yml))
publishes a single multi-arch tag matching the `# VERSION=` header in the
Dockerfile:

| Tag | Platforms |
|---|---|
| `ghcr.io/initialgyw/ansible-talos:1.13.5` | `linux/amd64`, `linux/arm64` |

## Pinned Tool Versions

The following versions are pinned via build args in the
[`Dockerfile`](./Dockerfile). `talosctl` **must** track the cluster's Talos
version (`group_vars/all.yaml` `talos_version`), because the `gen config`
templates are version-specific.

| Build arg | Default |
|---|---|
| `TALOSCTL_VERSION` | `v1.13.5` |
| `ANSIBLE_CORE_VERSION` | `2.21.1` |
| `GYWADMIN_OCI_VERSION` | `0.1.1` |

## Usage

The entrypoint is `ansible-playbook` (default `CMD` is `--version`), and the
working directory is `/workspace/ansible`. The `misc-scripts/ansible-talos.py`
wrapper invokes this image with the homelab repo mounted at `/workspace`; the
playbooks call the Vault helper via the `manage-vault` console script on `PATH`
(see `group_vars: manage_vault_cmd`).

Run a playbook by mounting the homelab repo at `/workspace`:

```sh
docker run --rm -it \
  -v ${PWD}:/workspace \
  ghcr.io/initialgyw/ansible-talos:1.13.5 \
  site.yaml
```

Override the entrypoint to run a Vault helper directly:

```sh
docker run --rm --entrypoint manage-vault \
  ghcr.io/initialgyw/ansible-talos:1.13.5 \
  list-secrets
```

## Building Locally

Override any pinned tool at build time with `--build-arg`, e.g.
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
