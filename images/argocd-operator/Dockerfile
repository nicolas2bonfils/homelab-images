# argocd-operator (argoproj-labs), multi-arch (the upstream image is amd64
# only). Same as upstream's Dockerfile, cross-compiled on the build
# platform (CGO off, as upstream): no emulation.
FROM --platform=$BUILDPLATFORM docker.io/library/golang:1.25 AS builder
ARG VERSION
ARG TARGETOS
ARG TARGETARCH
RUN git clone --depth 1 --branch ${VERSION} https://github.com/argoproj-labs/argocd-operator /src
WORKDIR /src
RUN CGO_ENABLED=0 GOOS=$TARGETOS GOARCH=$TARGETARCH go build -trimpath \
      -ldflags="-X github.com/argoproj-labs/argocd-operator/version.Version=${VERSION#v}" \
      -o /out/manager cmd/main.go

FROM gcr.io/distroless/static:nonroot
WORKDIR /
COPY --from=builder /out/manager .
COPY --from=builder /src/build/redis /var/lib/redis
USER 65532:65532
ENTRYPOINT ["/manager"]
