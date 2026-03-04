ARG VERSION
FROM golang:${VERSION}-alpine3.22

RUN apk update && \
    apk add --no-cache \
        make \
        gcc \
        g++ \
        git \
        openssh-client \
        docker \
        docker-cli-buildx \
        curl \
        binutils-gold \
        build-base \
        musl-dev && \
    sh -c "$(curl --location https://taskfile.dev/install.sh)" -- -d -b ~/.local/bin && \
    rm -rf /var/cache/apk/*