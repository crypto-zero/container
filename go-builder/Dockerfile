ARG VERSION
FROM golang:${VERSION}-alpine3.20

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
    rm -rf /var/cache/apk/*