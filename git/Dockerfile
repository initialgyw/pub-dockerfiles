# VERSION=3.22.2
FROM alpine:3.22.2

LABEL org.opencontainers.image.source=https://github.com/initialgyw/pub-dockerfiles
LABEL org.opencontainers.image.title="git"
LABEL org.opencontainers.image.description="Built from /git/Dockerfile in pub-dockerfiles"
LABEL org.opencontainers.image.documentation=https://github.com/initialgyw/pub-dockerfiles/blob/main/git/README.md
LABEL org.opencontainers.image.version="3.22.2"

RUN apk add --no-cache openssh-client git

ENTRYPOINT ["git"]
