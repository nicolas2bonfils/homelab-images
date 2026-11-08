# autopush-rs (mozilla-services), one crate per image. There is no public
# image: upstream's docker-compose builds from a checkout and its CI pushes
# to an internal GAR, so a local `podman build` of 30 minutes used to be the
# only way to get one.
#
# Differences from upstream's Dockerfile, all deliberate:
#   - the source is a checkout of VERSION, a commit, not an ADD of the
#     build context (GitHub Actions has no checkout of another repository);
#   - the two bases are pinned by digest, so a retagged rust:1.96-bookworm
#     cannot change the compiler under us;
#   - the redis feature is the default here: it is what talks to Valkey.
# amd64 only: the crates do not cross-compile cleanly (grpcio, cmake) and
# no cortex node is arm64.
FROM rust:1.96-bookworm@sha256:a339861ae23e9abb272cea45dfafde21760d2ce6577a70f8a926153677902663 AS builder
ARG VERSION
ARG CRATE
ARG BUILD_ARGS="--no-default-features --features redis"
ENV PATH=$PATH:/root/.cargo/bin

# cmake and clang are required by grpcio and google-cloud-rust, as upstream.
RUN apt-get -qq update && \
    apt-get -qq install --no-install-recommends -y cmake clang git ca-certificates && \
    rm -rf /var/lib/apt/lists

# A commit, not a tag: fetch it directly, and fall back to a full fetch if
# the server refuses to serve a bare sha.
RUN git init -q /app && cd /app && \
    git remote add origin https://github.com/mozilla-services/autopush-rs && \
    { git fetch -q --depth 1 origin "${VERSION}" && git checkout -q FETCH_HEAD; } || \
    { git fetch -q origin '+refs/heads/*:refs/remotes/origin/*' && git checkout -q "${VERSION}"; }

WORKDIR /app

RUN cargo --version && \
    rustc --version && \
    mkdir -m 755 bin && \
    cargo install --path "$CRATE" $BUILD_ARGS --locked --root /app

FROM debian:bookworm-slim@sha256:7ff6ba31527b9e336162fa061746c366c4b2032f4d2cff2078bea43f78efe3f8
ARG BINARY
RUN groupadd --gid 10001 app && \
    useradd --uid 10001 --gid 10001 --home /app --create-home app && \
    apt-get -qq update && \
    apt-get -qq install -y libssl-dev ca-certificates && \
    rm -rf /var/lib/apt/lists

COPY --from=builder /app/bin /app/bin
COPY --from=builder /app/version.json /app
COPY --from=builder /app/entrypoint.sh /app

WORKDIR /app
# ARG values are not available at runtime; upstream keeps running as root
# for the same reason (their `USER app` is commented out).
ENV BINARY=/app/bin/${BINARY}
ENTRYPOINT ["/app/entrypoint.sh"]
