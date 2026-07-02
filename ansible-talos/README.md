# ansible-talos

The credential-injecting container image that runs the Talos automation. It bundles `ansible-core` + `talosctl` (pinned to the cluster's Talos version).

- **Image:** `ghcr.io/initialgyw/ansible-talos`
- **Version:** `1.13.5` (equals the cluster's Talos version; see the
  `# VERSION=` header in the [`Dockerfile`](./Dockerfile))
- **Source:** [`ansible-talos/Dockerfile`](./Dockerfile)


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

## Usage

The entrypoint is `ansible-playbook` (default `CMD` is `--version`), and the
working directory is `/workspace/ansible`. The `misc-scripts/ansible-talos.py` wrapper invokes this image with the homelab repo mounted at `/workspace`.

Run a playbook by mounting the homelab repo at `/workspace`:

```sh
docker run --rm -it \
  -v ${PWD}:/workspace \
  ghcr.io/initialgyw/ansible-talos:1.13.5 \
  site.yaml
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
