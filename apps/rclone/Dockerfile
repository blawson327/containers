FROM ghcr.io/onedr0p/alpine:rolling@sha256:3fbc581cb0fe29830376161ae026e2a765dcc11e1747477fe9ebf155720b8638

ARG TARGETPLATFORM
ARG VERSION
ARG CHANNEL

#hadolint ignore=DL3018
RUN apk add --no-cache ca-certificates pigz rclone~"${VERSION}" tar zstd
#hadolint ignore=DL3059
RUN chown -R root:root /usr/bin/rclone \
    && chmod -R 755 /usr/bin/rclone \
    && ln -s /usr/bin/rclone /app/rclone \
    && rm -rf /tmp/*

CMD ["/app/rclone"]

LABEL org.opencontainers.image.source="https://github.com/rclone/rclone"
