# Kurento Media Server
#
# This Docker image is used to run an instance of Kurento Media Server.
# JSON-RPC API interface is exposed on port 8888.
#
#
# Build command
# -------------
#
# docker build [Args...] --tag kurento/kurento-media-server .
#
#
# Build arguments
# ---------------
#
# --build-arg UBUNTU_VERSION=<UbuntuVersion>
#
#   <UbuntuVersion> is like "xenial", "bionic", etc.
#   Valid values are listed here: https://hub.docker.com/_/buildpack-deps/
#
#   Optional. Default: "xenial".
#
# --build-arg KMS_VERSION=<KmsVersion>
#
#   <KmsVersion> is like "6.7.2", "6.9.0", etc.
#   Alternatively, "dev" is used to build a nightly version of KMS.
#
#   Required. Default: "0.0.0" (invalid version).
#
#
# Run command
# -----------
#
# docker run -d --name kms -p 8888:8888 kurento/kurento-media-server



ARG UBUNTU_VERSION=xenial

FROM ubuntu:${UBUNTU_VERSION}

MAINTAINER Patxi Gortázar <patxi.gortazar@gmail.com>
MAINTAINER Fede Diaz <nordri@gmail.com>
MAINTAINER Juan Navarro <juan.navarro@gmx.es>

ARG UBUNTU_VERSION=xenial
ARG KMS_VERSION=0.0.0

# Configure environment
# * LANG: Set the default locale for all commands
# * DEBIAN_FRONTEND: Disable user-facing questions and messages
ENV LANG=C.UTF-8 \
    DEBIAN_FRONTEND=noninteractive

# Install GnuPG, needed for `apt-key adv` (since Ubuntu 18.04)
RUN apt-get update \
 && apt-get install --yes \
        gnupg \
 && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install curl, needed for health check
RUN apt-get update \
 && apt-get install --yes \
        curl \
 && apt-get clean && rm -rf /var/lib/apt/lists/*

# Configure apt-get
# * Disable installation of recommended and suggested packages
# * Add Kurento package repository
RUN echo 'APT::Install-Recommends "false";' >/etc/apt/apt.conf.d/00recommends \
 && echo 'APT::Install-Suggests "false";' >>/etc/apt/apt.conf.d/00recommends \
 && echo "UBUNTU_VERSION=${UBUNTU_VERSION}" \
 && echo "KMS_VERSION=${KMS_VERSION}" \
 && echo "Apt source line: deb [arch=amd64] http://ubuntu.openvidu.io/${KMS_VERSION} ${UBUNTU_VERSION} kms6" \
 && echo "deb [arch=amd64] http://ubuntu.openvidu.io/${KMS_VERSION} ${UBUNTU_VERSION} kms6" >/etc/apt/sources.list.d/kurento.list \
 && apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 5AFA7A83

# Install Kurento Media Server
RUN apt-get update \
 && apt-get install --yes \
        kurento-media-server \
 ; apt-get clean && rm -rf /var/lib/apt/lists/*

# Install additional modules
# These might not be available in Ubuntu 18.04 (Bionic)
RUN apt-get update \
 && apt-get install --yes \
        kms-chroma \
        kms-crowddetector \
        kms-platedetector \
        kms-pointerdetector \
 || true \
 ; apt-get clean && rm -rf /var/lib/apt/lists/*

EXPOSE 8888

# Configure environment for KMS
# * Use default suggested logging levels:
#   https://doc-kurento.readthedocs.io/en/latest/features/logging.html#suggested-levels
# * Disable colors in debug logs
ENV GST_DEBUG="3,Kurento*:4,kms*:4,sdp*:4,webrtc*:4,*rtpendpoint:4,rtp*handler:4,rtpsynchronizer:4,agnosticbin:4" \
    GST_DEBUG_NO_COLOR=1

COPY ./entrypoint.sh /entrypoint.sh
COPY ./healthchecker.sh /healthchecker.sh

HEALTHCHECK --start-period=15s --interval=30s --timeout=3s --retries=1 CMD /healthchecker.sh

ENTRYPOINT ["/entrypoint.sh"]
