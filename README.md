# pub-dockerfiles

Public container images published to
[GHCR](https://github.com/initialgyw?tab=packages) under
`ghcr.io/initialgyw`. Each image lives in its own directory with a dedicated
README; images are built and pushed by the
[Docker CI workflow](./.github/workflows/docker.yml) whenever a component's
directory changes on `main`.

## Images

| Image | Version | Docs | Source |
|---|---|---|---|
| `ghcr.io/initialgyw/git` | `3.22.2` | [git/README.md](./git/README.md) | [git/Dockerfile](./git/Dockerfile) |
| `ghcr.io/initialgyw/opentofu` | `1.12.0` | [opentofu/README.md](./opentofu/README.md) | [opentofu/Dockerfile](./opentofu/Dockerfile) |
| `ghcr.io/initialgyw/ansible-talos` | `1.13.6.2` | [ansible-talos/README.md](./ansible-talos/README.md) | [ansible-talos/Dockerfile](./ansible-talos/Dockerfile) |

## Conventions

- Each `Dockerfile` declares its image version in a `# VERSION=` header on the
  first line. CI reads this header and publishes a single multi-arch
  (`linux/amd64`, `linux/arm64`) tag `ghcr.io/initialgyw/<component>:<VERSION>`.
- Each image carries OCI labels, including
  `org.opencontainers.image.documentation` pointing at its README and
  `org.opencontainers.image.version` matching the `# VERSION=` header.
- See each image's README for usage and local build instructions.
