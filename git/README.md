# How to use

```sh

docker run --rm -it -v ${PWD}:/git -v $SSH_AUTH_SOCK:/ssh-agent -e SSH_AUTH_SOCK=/ssh-agent -w /git ghcr.io/initialgyw/git:latest-amd64 clone REPO
```
