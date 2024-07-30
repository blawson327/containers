FROM docker.io/library/ubuntu:24.04

ARG TARGETPLATFORM
ARG VERSION

# Sets Jackett config directory
# https://github.com/Jackett/Jackett#home-directory-1
ENV XDG_CONFIG_HOME=/config

ENV UMASK="0002" \
    TZ="Etc/UTC"

USER root

RUN echo "TARGETPLATFORM is ${TARGETPLATFORM}"
RUN echo "VERSION is ${VERSION}"
# hadolint ignore=DL3008,DL3015,SC2086
RUN \
  apt-get -qq update \
  && \
  apt-get -qq install -y \
    curl \
    libicu-dev \
  && \
  case "${TARGETPLATFORM}" in \
    'linux/amd64') export ARCH='AMDx64' ;; \
    'linux/arm64') export ARCH='ARM64' ;; \
    'linux/arm/v7') export ARCH='ARM32' ;; \
  esac \
  && \
  mkdir -p /app \
  && \
  mkdir -p /config \
  && \
  curl -fsSL -o /tmp/jackett.tar.gz \
    "https://github.com/Jackett/Jackett/releases/download/v${VERSION}/Jackett.Binaries.Linux${ARCH}.tar.gz" \
  && tar ixzf /tmp/jackett.tar.gz -C /app --strip-components 1 \
  && \
  rm -rf \
    /app/JackettUpdater* \
  && \
  apt-get autoremove -y \
  && apt-get clean \
  && \
  rm -rf \
    /tmp/* \
    /var/lib/apt/lists/* \
    /var/tmp/ \
  && chmod -R 755 /app \
  && chmod -R 777 /config \
  && printf "umask %d" "${UMASK}" >> /etc/bash.bashrc

USER nobody:nogroup

EXPOSE 9117

COPY ./apps/jackett/entrypoint.sh /entrypoint.sh
CMD ["/entrypoint.sh"]

LABEL \
  org.opencontainers.image.title="Jackett" \
  org.opencontainers.image.source="https://github.com/Jackett/Jackett" \
  org.opencontainers.image.version="${VERSION}"
