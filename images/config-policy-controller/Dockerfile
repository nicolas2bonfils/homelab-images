# config-policy-controller of Open Cluster Management, multi-arch (the upstream image is
# amd64 only). Same binary and entrypoint as upstream's build/Dockerfile,
# cross-compiled in pure Go (CGO off) on the build platform: no emulation.
FROM --platform=$BUILDPLATFORM docker.io/library/golang:1.25 AS builder
ARG VERSION
ARG TARGETOS
ARG TARGETARCH
RUN git clone --depth 1 --branch ${VERSION} https://github.com/open-cluster-management-io/config-policy-controller /src
WORKDIR /src
RUN CGO_ENABLED=0 GOOS=$TARGETOS GOARCH=$TARGETARCH go build -trimpath -o /out/config-policy-controller ./

FROM registry.access.redhat.com/ubi9/ubi-minimal:9.8
ENV OPERATOR=/usr/local/bin/config-policy-controller \
    USER_UID=1001 \
    USER_NAME=config-policy-controller
COPY --from=builder /out/config-policy-controller ${OPERATOR}
COPY --from=builder /src/build/bin /usr/local/bin
# Upstream runs build/bin/user_setup (a passwd entry for 1001); a numeric
# user needs none.
USER 1001
ENTRYPOINT ["/usr/local/bin/entrypoint", "controller"]
