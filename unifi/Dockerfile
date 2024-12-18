ARG BUILD_FROM=ubuntu:noble
# hadolint ignore=DL3006
FROM ${BUILD_FROM}

# Environment variables
ENV \
    CARGO_NET_GIT_FETCH_WITH_CLI=true \
    DEBIAN_FRONTEND="noninteractive" \
    HOME="/root" \
    LANG="C.UTF-8" \
    PIP_DISABLE_PIP_VERSION_CHECK=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_PREFER_BINARY=1 \
    PS1="$(whoami)@$(hostname):$(pwd)$ " \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    S6_BEHAVIOUR_IF_STAGE2_FAILS=2 \
    S6_CMD_WAIT_FOR_SERVICES_MAXTIME=0 \
    S6_CMD_WAIT_FOR_SERVICES=1 \
    YARN_HTTP_TIMEOUT=10000
    
# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Setup base system
ARG BUILD_ARCH=amd64
ARG S6_OVERLAY_VERSION="3.2.0.2"
RUN \
    apt-get update \
    && apt-get upgrade -y \
    && apt-get install -y --no-install-recommends \
        binutils=2.34-6ubuntu1.9 \
        libcap2=1:2.32-1ubuntu0.1 \
        logrotate=3.14.0-4ubuntu3 \
        mongodb-server=1:3.6.9+really3.6.8+90~g8e540c0b6d-0ubuntu5.3 \
        openjdk-17-jre-headless=17* \
        ca-certificates=20240203 \
        curl=8.5.0-2ubuntu10.5 \
        jq=1.7.1-3build1 \
        tzdata=2024a-2ubuntu1 \
        xz-utils=5.6.1+really5.4.5-1 \
    \
    && S6_ARCH="${BUILD_ARCH}" \
    && if [ "${BUILD_ARCH}" = "i386" ]; then S6_ARCH="i686"; \
    elif [ "${BUILD_ARCH}" = "amd64" ]; then S6_ARCH="x86_64"; \
    elif [ "${BUILD_ARCH}" = "armv7" ]; then S6_ARCH="arm"; fi \
    \
    && curl -L -s "https://github.com/just-containers/s6-overlay/releases/download/v${S6_OVERLAY_VERSION}/s6-overlay-noarch.tar.xz" \
        | tar -C / -Jxpf - \
    \
    && curl -L -s "https://github.com/just-containers/s6-overlay/releases/download/v${S6_OVERLAY_VERSION}/s6-overlay-${S6_ARCH}.tar.xz" \
        | tar -C / -Jxpf - \
    \
    && curl -L -s "https://github.com/just-containers/s6-overlay/releases/download/v${S6_OVERLAY_VERSION}/s6-overlay-symlinks-noarch.tar.xz" \
        | tar -C / -Jxpf - \
    \
    && curl -L -s "https://github.com/just-containers/s6-overlay/releases/download/v${S6_OVERLAY_VERSION}/s6-overlay-symlinks-arch.tar.xz" \
        | tar -C / -Jxpf - \
    \
    && mkdir -p /etc/fix-attrs.d \
    && mkdir -p /etc/services.d \
    \
    && curl -J -L -o /tmp/unifi.deb \
        "https://dl.ui.com/unifi/8.6.9/unifi_sysvinit_all.deb" \
    \
    && dpkg --install /tmp/unifi.deb \
    && apt-get purge -y --auto-remove \
        xz-utils \
    && apt-get clean \
    && rm -fr \
        /tmp/* \
        /var/cache/* \
        /var/lib/apt/lists/* \
        /var/log/*.log \
        /var/log/apt

# Copy root filesystem
COPY rootfs /

# Copy s6-overlay adjustments
COPY s6-overlay /package/admin/s6-overlay-${S6_OVERLAY_VERSION}/

# Entrypoint & CMD
ENTRYPOINT [ "/init" ]

# Health check
HEALTHCHECK --start-period=5m \
    CMD curl --insecure --fail https://localhost:8443 || exit 1
