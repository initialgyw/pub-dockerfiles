# pub-dockerfiles

## opentofu

### Building and tagging image

```bash
# Build for multiple platforms
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t ghcr.io/initialgyw/opentofu:1.12.0-alpine-3.23.4 \
  -t ghcr.io/initialgyw/opentofu:1.12.0-alpine \
  -t ghcr.io/initialgyw/opentofu:1.12-alpine \
  -t ghcr.io/initialgyw/opentofu:alpine \
  -f opentofu/Dockerfile \
  --push opentofu/
```

## ansible-talos

The credential-injecting container that runs the Talos automation. It bundles
everything the playbooks need so the ONLY persisted cluster state lives in OCI
Vault and the working tree stays clean:

- **ansible-core** — the playbook engine.
- **talosctl** — PINNED to `v1.13.5` (the cluster's Talos version; `gen config`
  templates are version-specific, so this pin tracks
  `ansible/group_vars/all.yaml` `talos_version`).
- **kubectl**, **helm** (v3) — used by the roles for node-Ready / Cilium steps.
- **python3 + the `oci` SDK** — plus `cryptography` / `pyyaml`, the deps of the
  OCI Vault helper.
- **`misc-scripts/oci/manage-vault.py` + `_oci_common.py`** — the ONLY interface
  to OCI Vault (no `oci` CLI anywhere).

The image is invoked by the [`misc-scripts/ansible-talos.py`](../../misc-scripts/ansible-talos.py)
wrapper in the `gywadmin-homelab` repo, which injects OCI credentials, mounts the repo,
and wipes an ephemeral OCI config dir on exit.

### Intended tag

```
ghcr.io/initialgyw/ansible-talos:1.13.5
```

The tag equals the Talos version so the talosctl pin is obvious. Analogous to
`ghcr.io/initialgyw/opentofu:1.12-alpine` used by `docker-tofu.py`.

### Build

The build context **must be the `gywadmin-homelab` repo root** so the `COPY misc-scripts/oci/...`
lines can reach the helper. (Assuming `pub-dockerfiles` and `gywadmin-homelab` are cloned side-by-side):

```bash
# Single-arch, local Apple-Silicon (arm64) host:
docker build -t ghcr.io/initialgyw/ansible-talos:1.13.5 \
  -f ../pub-dockerfiles/ansible-talos/Dockerfile .

# Multi-arch (arm64 + amd64), push to GHCR:
docker buildx build --platform linux/arm64,linux/amd64 \
  -t ghcr.io/initialgyw/ansible-talos:1.13.5 \
  -f ../pub-dockerfiles/ansible-talos/Dockerfile --push .
```

### Entrypoint

`ENTRYPOINT ["ansible-playbook"]` — the default. The wrapper passes the playbook
+ args after the `--` separator. To run the Vault helper directly (debugging),
override the entrypoint:

```bash
docker run --rm --entrypoint python3 \
  ghcr.io/initialgyw/ansible-talos:1.13.5 \
  /opt/oci/manage-vault.py list-secrets --oci-config-file /oci/config
```

### Networking

The container must reach BOTH the OCI Vault (public internet) and the Talos
nodes on the LAN. **The user owns container↔LAN networking**: pass extra
`docker run` args through the wrapper, e.g.

```bash
./misc-scripts/ansible-talos.py --network host -- misc-playbooks/talos/init-cluster.yaml ...
# or an arbitrary passthrough:
./misc-scripts/ansible-talos.py --docker-arg=--add-host=... -- ...
```

### Version pins

| Tool         | Pinned version | Where it tracks                              |
| ------------ | -------------- | -------------------------------------------- |
| talosctl     | `v1.13.5`      | `ansible/group_vars/all.yaml` `talos_version` |
| kubectl      | `v1.36.2`      | current stable                               |
| helm         | `v3.21.2`      | v3 semantics                                 |
| ansible-core | `2.21.1`       | current stable                               |
| `oci` SDK    | `2.181.0`      | current PyPI                                 |

Override any pin at build time with `--build-arg`, e.g.
`--build-arg TALOSCTL_VERSION=v1.13.6`.
