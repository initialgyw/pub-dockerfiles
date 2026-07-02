# git

A minimal [Alpine](https://alpinelinux.org/)-based image that ships `git` and
`openssh-client`, with `git` as the entrypoint.

- **Image:** `ghcr.io/initialgyw/git`
- **Version:** `3.22.2` (tracks the Alpine base version pinned in the
  [`Dockerfile`](./Dockerfile))
- **Source:** [`git/Dockerfile`](./Dockerfile)

## Tags

The CI workflow ([`.github/workflows/docker.yml`](../.github/workflows/docker.yml))
publishes a single multi-arch tag matching the `# VERSION=` header in the
Dockerfile:

| Tag | Platforms |
|---|---|
| `ghcr.io/initialgyw/git:3.22.2` | `linux/amd64`, `linux/arm64` |

## Usage

Clone a repository using your host's SSH agent and identity:

```sh
sudo docker run --rm -it \
  -v ${PWD}:/git \
  -v $SSH_AUTH_SOCK:/ssh-agent \
  -v /etc/passwd:/etc/passwd:ro \
  -v /etc/group:/etc/group:ro \
  -e SSH_AUTH_SOCK=/ssh-agent \
  -e HOME=/git \
  --user "$(id -u):$(id -g)" \
  -w /git \
  ghcr.io/initialgyw/git:3.22.2 \
  clone <REPO>
```

The entrypoint is `git`, so any `git` subcommand can be appended directly (for
example `status`, `pull`, `log`).

## Building Locally

```bash
# Build for multiple platforms and push to GHCR
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t ghcr.io/initialgyw/git:3.22.2 \
  -f git/Dockerfile \
  --push git/
```
