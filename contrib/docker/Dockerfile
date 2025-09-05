FROM alpine:3.22 AS builder

RUN apk add --no-cache \
    build-base \
    cmake \
    boost-dev \
    libevent-dev \
    sqlite-dev \
    zeromq-dev \
    coreutils \
    binutils

WORKDIR /opt/bitcoin

COPY . .

WORKDIR /opt/bitcoin/build

RUN cmake .. \
    -DCMAKE_INSTALL_PREFIX="/opt/bitcoin/build/" \
    -DBUILD_DAEMON="ON" \
    -DBUILD_CLI="ON" \
    -DENABLE_WALLET="OFF" \
    -DWITH_ZMQ="OFF" \
    -DBUILD_GUI="OFF" \
    -DBUILD_TESTS="OFF" \
    -DBUILD_BENCH="OFF" \
    -DBUILD_FUZZ_BINARY="OFF" \
    -DBUILD_UTIL_CHAINSTATE="OFF" \
    -DWITH_BDB="OFF" \
    -DWITH_USDT="OFF" \
    -DWITH_MULTIPROCESS="OFF" \
    -DINSTALL_MAN="OFF" \
    -DWITH_CCACHE="OFF" \
    -DAPPEND_CXXFLAGS="-O2" \
    -DAPPEND_LDFLAGS="-flto"

RUN cmake --build . --target install --parallel $(nproc)

RUN strip --strip-unneeded bin/*

FROM alpine:3.22 AS final

ARG USER_ID=1000
ARG GROUP_ID=1000

RUN apk add --no-cache \
    libevent \
    sqlite-libs \
    zeromq \
    boost-system \
    boost-filesystem \
    boost-program_options

COPY --from=builder /opt/bitcoin/build/bin/  /usr/local/bin/

RUN addgroup -S -g ${GROUP_ID} bitcoin && \
    adduser -S -u ${USER_ID} -G bitcoin -H -s /bin/false bitcoin

VOLUME ["/var/lib/bitcoin", "/etc/bitcoin/bitcoin.conf"]
EXPOSE 8333 8332
WORKDIR /var/lib/bitcoin
USER bitcoin

ENTRYPOINT ["bitcoind", "-conf=/etc/bitcoin/bitcoin.conf", "-datadir=/var/lib/bitcoin"]
