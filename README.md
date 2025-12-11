# Container Images

Docker images for building and running Go applications.

## Images

### go-builder
Build environment image with complete compilation toolchain.

**Included Tools**:
- Go compiler
- make, gcc, g++
- git, openssh-client
- docker, docker-cli-buildx
- curl, binutils-gold
- build-base, musl-dev

### go-runtime
Runtime image for Go applications, smaller size for deployment.

**Included Tools**:
- Go runtime
- gops (Go process diagnostics tool)
- Common utilities: curl, jq, gzip, bind-tools
- Network tools: proxychains-ng
- Timezone and certificate support

## Automated Builds

This project uses GitHub Actions to automatically build and publish images.

### Workflow Features

- **Automatic Version Detection**: Checks for the latest Go version from `go.dev`
- **Smart Building**: Only builds if the version doesn't exist in the registry
- **Multi-Architecture**: Builds for `linux/amd64` and `linux/arm64`
- **Daily Schedule**: Runs daily at 00:00 UTC
- **Manual Trigger**: Can be triggered manually via workflow_dispatch

### Image Tags

Images are published to GitHub Container Registry:

```
ghcr.io/<your-username>/go-builder:v1.23.4
ghcr.io/<your-username>/go-runtime:v1.23.4
```

### Usage

Pull and use the images:

```bash
# For building
FROM ghcr.io/<your-username>/go-builder:v1.23.4 AS builder
WORKDIR /build
COPY . .
RUN go build -o app

# For runtime
FROM ghcr.io/<your-username>/go-runtime:v1.23.4
COPY --from=builder /build/app /app/bin/app
ENV PROJECT_NAME=app
```

## Environment Variables

### go-runtime

- `TZ`: Set timezone (e.g. `Asia/Shanghai`)
- `CI_BUILD_INFO`: Build information
- `PROJECT_NAME`: Project executable name (required)

## Manual Build (Optional)

If you need to build locally for testing:

```bash
# Get the latest Go version first
LATEST_GO_VERSION=$(curl -s 'https://go.dev/VERSION?m=text' | head -n 1 | cut -c 3-)

# Build go-builder
docker build --build-arg VERSION=${LATEST_GO_VERSION} -t go-builder:${LATEST_GO_VERSION} ./go-builder

# Build go-runtime
docker build --build-arg VERSION=${LATEST_GO_VERSION} -t go-runtime:${LATEST_GO_VERSION} ./go-runtime

# Multi-architecture build
docker buildx build --platform linux/amd64,linux/arm64 \
  --build-arg VERSION=${LATEST_GO_VERSION} \
  -t your-registry/go-builder:${LATEST_GO_VERSION} \
  --push \
  ./go-builder
```

## Notes

1. **Version Strategy**: 
   - Dockerfiles do **not** have a default version - builds require explicit version specification
   - GitHub Actions automatically detects and builds with the latest Go version
   - **Local builds must specify `--build-arg VERSION=x.y.z`** - builds will fail without this parameter
   - This design forces version awareness and prevents accidentally using outdated versions

2. Images are based on Alpine Linux 3.20 (small size, uses musl libc, pinned to avoid busybox 1.37 bug)

3. go-runtime requires `PROJECT_NAME` environment variable pointing to the executable

4. Both images support multi-architecture (amd64/arm64)

5. **go-runtime design**: Uses pure `alpine:3.20` as base (not `golang:alpine`) for minimal runtime size (~50MB vs ~400MB)
