ARG VERSION=1.23.2
FROM golang:${VERSION}-alpine as builder
RUN  apk add --no-cache git
RUN go install github.com/google/gops@latest

ARG VERSION
FROM golang:${VERSION}-alpine

RUN mkdir -p /app/bin/ /app/configs /app/logs
WORKDIR /app
ENV PATH $PATH:/app/bin

COPY --from=builder /go/bin/gops /usr/bin/gops
COPY --from=builder /usr/local/go/bin/go /usr/bin/go
COPY docker-entrypoint /app/bin/
RUN chmod +x /app/bin/docker-entrypoint

## install deps
RUN apk update && \
    apk upgrade && \
    apk add tzdata ca-certificates proxychains-ng lzo-dev bind-tools gzip jq curl && \
    update-ca-certificates && \
    apk upgrade

# change timezone to asia/shanghai
# RUN echo "Asia/Shanghai" > /etc/timezone
# RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

CMD ["docker-entrypoint"]

