ARG BUILD_FROM
FROM $BUILD_FROM
FROM ${BUILD_FROM} as builder

ENV LANG C.UTF-8

# Setup base
ARG VZLOGGER_VERSION="0.8.8"
ARG LIBSML_VERSION="1.1.3"

# install packages
RUN apk add --no-cache \
    build-base \
    git \
    util-linux-dev \
    cmake \
    json-c-dev \
    libmicrohttpd-dev \
    openssl-dev \
    curl-dev \
    gnutls-dev \
    libgcrypt-dev \
    cyrus-sasl-dev \
    libunistring-dev \
    mosquitto-dev \
    autoconf \
    automake \
    libtool
    
WORKDIR /vzlogger

RUN git clone https://github.com/rscada/libmbus.git --depth 1
	
RUN mkdir -p /vzlogger/vzlogger && \
    curl -L -s "https://github.com/volkszaehler/vzlogger/archive/refs/tags/v${VZLOGGER_VERSION}.tar.gz" \
    | tar zxvf - --strip-components 1 -C /vzlogger/vzlogger
	
RUN mkdir -p /vzlogger/libsml && \
    curl -L -s "https://github.com/volkszaehler/libsml/archive/refs/tags/v${LIBSML_VERSION}.tar.gz" \
    | tar zxvf - --strip-components 1 -C /vzlogger/libsml
	
WORKDIR /vzlogger/libsml
RUN make install -C sml

WORKDIR /vzlogger/libmbus
RUN ./build.sh && make install

WORKDIR /vzlogger/vzlogger
RUN cmake -DBUILD_TEST=off && \
    make && make install

# Create final and small Image
FROM $BUILD_FROM

# Build arguments
ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_DESCRIPTION
ARG BUILD_NAME
ARG BUILD_REF
ARG BUILD_REPOSITORY
ARG BUILD_VERSION

# Labels
LABEL \
    io.hass.name="${BUILD_NAME}" \
    io.hass.description="${BUILD_DESCRIPTION}" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.url="https://github.com/${BUILD_REPOSITORY}/blob/main/README.md" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.documentation="https://github.com/${BUILD_REPOSITORY}/blob/main/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}

# libsml is linked statically => no need to copy
COPY --from=builder /usr/local/bin/vzlogger /usr/local/bin/vzlogger
COPY --from=builder /usr/local/lib/libmbus.so* /usr/local/lib/

RUN apk add --no-cache \
    libcurl \
    gnutls \
    libsasl \
    libuuid \
    openssl-dev \
    libgcrypt \
    libmicrohttpd \
    libltdl \
    libatomic \
    json-c \
    leptonica \
    mosquitto-dev \
    libunistring

# Copy data for add-on
COPY vzlogger.conf /
COPY run.sh /
RUN chmod a+x /run.sh

CMD [ "/run.sh" ]
