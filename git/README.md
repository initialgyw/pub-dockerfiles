# How to use

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
  ghcr.io/initialgyw/git:latest-amd64 \
  clone <REPO>
```
