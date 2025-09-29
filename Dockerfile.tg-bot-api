FROM alpine:3.22 AS builder
ARG JOBS=4
RUN apk add --no-cache \
    git cmake g++ make gperf \
    openssl-dev zlib-dev \
    ca-certificates linux-headers
WORKDIR /src
RUN git clone --depth 1 --recursive https://github.com/tdlib/telegram-bot-api.git .
RUN mkdir -p build && cd build && \
    cmake -DCMAKE_BUILD_TYPE=Release \
          -DCMAKE_CXX_FLAGS="-Os" \
          -DCMAKE_EXE_LINKER_FLAGS="-s" \
          -DCMAKE_INSTALL_PREFIX=/usr/local .. && \
    cmake --build . -j"${JOBS}" && \
    cmake --install . && \
    strip /usr/local/bin/telegram-bot-api

FROM alpine:3.22

ARG GIT_COMMIT=unknown
ARG BUILD_DATE=unknown

# metadata labels
LABEL org.opencontainers.image.title="Telegram Bot API Server" \
      org.opencontainers.image.description="Minimal Alpine-based container for tdlib/telegram-bot-api server." \
      org.opencontainers.image.url="https://github.com/tdlib/telegram-bot-api" \
      org.opencontainers.image.source="https://github.com/${GITHUB_REPOSITORY}" \
      org.opencontainers.image.licenses="MIT" \
      org.opencontainers.image.authors="Troublescope <121922135+troublescope@users.noreply.github.com>"
      
RUN apk add --no-cache libstdc++ zlib openssl ca-certificates busybox
COPY --from=builder /usr/local/bin/telegram-bot-api /usr/local/bin/telegram-bot-api
RUN ln -sf /bin/busybox /usr/bin/wget

WORKDIR /var/lib/telegram-bot-api
ENV DATA_DIR=/data \
    TMP_DIR=/tmp-data \
    HTTP_PORT=8081 \
    LOG_VERBOSITY=1 \
    MAX_CONNECTIONS=100

RUN mkdir -p /data /tmp-data
USER 65532:65532

ENTRYPOINT ["/bin/sh","-c","\
  test -n \"$TELEGRAM_API_ID\" && test -n \"$TELEGRAM_API_HASH\" || { echo 'Set TELEGRAM_API_ID and TELEGRAM_API_HASH'; exit 1; }; \
  exec /usr/local/bin/telegram-bot-api \
    --api-id=$TELEGRAM_API_ID \
    --api-hash=$TELEGRAM_API_HASH \
    --http-port=${HTTP_PORT:-8081} \
    --dir=${DATA_DIR:-/data} \
    --temp-dir=${TMP_DIR:-/tmp-data} \
    --verbosity=${LOG_VERBOSITY:-1} \
    --max-webhook-connections=${MAX_CONNECTIONS:-100} \
    ${LOCAL_MODE:+--local} \
    ${FILTER:+--filter=$FILTER} \
"]

EXPOSE 8081

HEALTHCHECK --interval=30s --timeout=5s --start-period=15s \
  CMD /usr/bin/wget -qO- http://127.0.0.1:${HTTP_PORT:-8081}/ >/dev/null || exit 1
