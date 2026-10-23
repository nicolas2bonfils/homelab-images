# governance-policy-framework-addon of Open Cluster Management, multi-arch (the upstream image is
# amd64 only). Same binary and entrypoint as upstream's build/Dockerfile,
# cross-compiled in pure Go (CGO off) on the build platform: no emulation.
FROM --platform=$BUILDPLATFORM docker.io/library/golang:1.25 AS builder
ARG VERSION
ARG TARGETOS
ARG TARGETARCH
RUN git clone --depth 1 --branch ${VERSION} https://github.com/open-cluster-management-io/governance-policy-framework-addon /src
WORKDIR /src
RUN CGO_ENABLED=0 GOOS=$TARGETOS GOARCH=$TARGETARCH go build -trimpath -o /out/governance-policy-framework-addon ./

FROM registry.access.redhat.com/ubi9/ubi-minimal:9.8
ENV OPERATOR=/usr/local/bin/governance-policy-framework-addon \
    USER_UID=1001 \
    USER_NAME=governance-policy-framework-addon
COPY --from=builder /out/governance-policy-framework-addon ${OPERATOR}
COPY --from=builder /src/build/bin /usr/local/bin
# Upstream runs build/bin/user_setup (a passwd entry for 1001); a numeric
# user needs none.
USER 1001
ENTRYPOINT ["/usr/local/bin/entrypoint"]
