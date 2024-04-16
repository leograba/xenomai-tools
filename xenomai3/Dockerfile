# xenomai release
# 3.2.4 is the latest as of April 15th 2024
ARG XENOMAI_RELEASE="v3.2.4"

# libs staging directory
ARG XENOMAI_STAGING_DIR="/xenomai/staging"

# Build in a separate stage, so build-time dependencies and source are
# not included in the final container image
FROM ubuntu:22.04 AS build

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
FROM ubuntu:22.04 AS deploy

ARG XENOMAI_STAGING_DIR

# Install runtime dependencies and useful tools
RUN apt-get update && apt-get install -y \
    rt-tests \
    libatomic1 && \
    rm -rf /var/lib/apt/lists/*

# Copy the build artifacts from the staging directory
COPY --from=build ${XENOMAI_STAGING_DIR}/usr /usr
