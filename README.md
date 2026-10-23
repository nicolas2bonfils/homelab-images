# homelab-images

Multi-arch (linux/amd64, linux/arm64) builds of upstream container images
that are published for amd64 only, needed on the homelab's arm64 machines.

| Image | Upstream | Versions |
|---|---|---|
| `config-policy-controller` | open-cluster-management-io/config-policy-controller | v0.18.0, v0.19.0 |
| `governance-policy-framework-addon` | open-cluster-management-io/governance-policy-framework-addon | v0.18.0, v0.19.0 |
| `argocd-operator` | argoproj-labs/argocd-operator | v0.18.0 |

Each `images/<image>/Dockerfile` clones the upstream tag given as
`VERSION` and builds the same binary and entrypoint as the upstream
Dockerfile, cross-compiled in pure Go on the build platform (no
emulation). The workflow pushes `ghcr.io/<owner>/<image>:<tag>`.

Local build: `podman build --platform linux/arm64 --build-arg VERSION=v0.18.0 images/<image>`.

These images exist only until upstream publishes arm64 builds.
