# argocd-pull-integration (OCM's argocd-agent addon) built against the
# multi-arch argocd-operator of this repository: the addon's agent deploys
# the argocd-operator image compiled into it (main.operatorImage), the
# upstream one being amd64 only. Same as upstream's Dockerfile otherwise,
# cross-compiled on the build platform.
FROM --platform=$BUILDPLATFORM docker.io/library/golang:1.26 AS builder
ARG VERSION
ARG TARGETOS
ARG TARGETARCH
ARG ARGOCD_OPERATOR_IMAGE=ghcr.io/nicolas2bonfils/argocd-operator:v0.18.0
RUN git clone --depth 1 --branch ${VERSION} https://github.com/open-cluster-management-io/argocd-pull-integration /src
WORKDIR /src
RUN CGO_ENABLED=0 GOOS=$TARGETOS GOARCH=$TARGETARCH go build -trimpath \
      -ldflags "-X main.operatorImage=${ARGOCD_OPERATOR_IMAGE}" \
      -o /out/manager cmd/main.go

FROM gcr.io/distroless/static:nonroot
WORKDIR /
COPY --from=builder /out/manager .
USER 65532:65532
ENTRYPOINT ["/manager"]
