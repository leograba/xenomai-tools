# container release
# 24.04 is the latest Ubuntu as of May 3rd 2024
ARG CONTAINER_RELEASE="22.04"

# libevl release
# r50 is the latest as of Feb 7th 2025
ARG EVL_RELEASE="r50"

# Kernel headers because libevl requires it
# building for LTS kernel 6.6.y as of Feb 7th 2025
ARG LINUX_HEADERS_VERSION="v6.6.69-evl1-rebase"

# libevl staging directory
ARG EVL_STAGING_DIR="/libevl/staging"

# Build in a separate stage, so build-time dependencies and source are
# not included in the final container image
FROM ubuntu:${CONTAINER_RELEASE} AS build

ARG EVL_RELEASE
ARG LINUX_HEADERS_VERSION
ARG EVL_STAGING_DIR

# Install build dependencies
RUN apt-get update && apt-get install -y \
    wget \
    build-essential \
    pkg-config \
    cmake \
    meson \
    ninja-build \
    libbpf-dev \
    && \
    rm -rf /var/lib/apt/lists/*

# Download linux-evl source due to required UAPI headers
WORKDIR /linux
RUN wget -q "https://source.denx.de/Xenomai/xenomai4/linux-evl/-/archive/${LINUX_HEADERS_VERSION}/linux-evl-${LINUX_HEADERS_VERSION}.tar.bz2" && \
    tar xvf "linux-evl-${LINUX_HEADERS_VERSION}.tar.bz2"

# Download libevl source
WORKDIR /libevl
RUN wget -q "https://source.denx.de/Xenomai/xenomai4/libevl/-/archive/${EVL_RELEASE}/libevl-${EVL_RELEASE}.tar.bz2" && \
    tar xvf "libevl-${EVL_RELEASE}.tar.bz2"

# Build and install to staging directory
RUN mkdir build && cd build && \
    meson setup \
    -Duapi="/linux/linux-evl-${LINUX_HEADERS_VERSION}" \
    -Dbuildtype=release \
    -Dprefix=${EVL_STAGING_DIR} \
    . \
    "/libevl/libevl-${EVL_RELEASE}" && \ 
    meson compile && \
    ninja install

# Deploy only the build output to the final container image
FROM ubuntu:${CONTAINER_RELEASE} AS deploy

ARG EVL_STAGING_DIR

# Install runtime dependencies and useful tools
RUN apt-get update && apt-get install -y \
    rt-tests \
    libatomic1 \
    libbpf0 \
    && \
    rm -rf /var/lib/apt/lists/*

# Copy the build artifacts from the staging directory
COPY --from=build ${EVL_STAGING_DIR} /usr/
