# libevl release
# r47 is the latest as of March 15th 2024
ARG EVL_RELEASE="r47"

# Kernel headers because libevl requires it
# building for SLTS kernel 5.10.y as of March 15th 2024
ARG LINUX_HEADERS_VERSION="v5.10.209-evl1-rebase"

# libevl staging directory
ARG EVL_STAGING_DIR="/libevl/staging"

# Build in a separate stage, so build-time dependencies and source are
# not included in the final container image
FROM ubuntu:22.04 AS build

ARG EVL_RELEASE
ARG LINUX_HEADERS_VERSION
ARG EVL_STAGING_DIR

# Install build dependencies
RUN apt-get update && apt-get install -y \
    wget \
    build-essential \
    meson \
    ninja-build && \
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
FROM ubuntu:22.04 AS deploy

ARG EVL_STAGING_DIR

# Install runtime dependencies and useful tools
RUN apt-get update && apt-get install -y \
    rt-tests \
    libatomic1 && \
    rm -rf /var/lib/apt/lists/*

# Copy the build artifacts from the staging directory
COPY --from=build ${EVL_STAGING_DIR} /usr/
