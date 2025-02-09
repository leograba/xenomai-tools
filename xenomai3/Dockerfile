# container release
# 24.04 is the latest Ubuntu as of May 3rd 2024
ARG CONTAINER_RELEASE="24.04"

# xenomai release
# 3.2.6 is the latest as of February 9th 2025
ARG XENOMAI_RELEASE="v3.2.6"

# libs staging directory
ARG XENOMAI_STAGING_DIR="/xenomai/staging"

# Build in a separate stage, so build-time dependencies and source are
# not included in the final container image
FROM ubuntu:${CONTAINER_RELEASE} AS build

ARG XENOMAI_RELEASE
ARG LINUX_HEADERS_VERSION
ARG XENOMAI_STAGING_DIR

# Install build dependencies
RUN apt-get update && apt-get install -y \
    wget \
    build-essential \
    libtool \
    autoconf \
    automake \
    autotools-dev \
    pkg-config && \
    rm -rf /var/lib/apt/lists/*

# Download xenomai source
WORKDIR /xenomai
RUN wget -q "https://source.denx.de/Xenomai/xenomai/-/archive/${XENOMAI_RELEASE}/xenomai-${XENOMAI_RELEASE}.tar.bz2" && \
    tar xvf "xenomai-${XENOMAI_RELEASE}.tar.bz2" && \
    mv xenomai-${XENOMAI_RELEASE} xenomai

# Build and install to staging directory
WORKDIR /xenomai/xenomai
RUN ./scripts/bootstrap && \
    mkdir build && cd build && \
    ../configure \
        --prefix=/usr \
        --includedir=/usr/include/xenomai \
        --with-core=cobalt \
        --enable-smp \
        --enable-pshared && \
    make DESTDIR=${XENOMAI_STAGING_DIR} install

# Deploy only the build output to the final container image
FROM ubuntu:${CONTAINER_RELEASE} AS deploy

ARG XENOMAI_STAGING_DIR

# Install runtime dependencies and useful tools
RUN apt-get update && apt-get install -y \
    rt-tests \
    libatomic1 && \
    rm -rf /var/lib/apt/lists/*

# Copy the build artifacts from the staging directory
COPY --from=build ${XENOMAI_STAGING_DIR}/usr /usr
