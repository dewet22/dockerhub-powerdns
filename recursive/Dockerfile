FROM alpine AS builder

ARG RECURSOR_VERSION

RUN apk --update upgrade && \
    apk add ca-certificates curl jq && \
    apk add --virtual .build-depends \
      file gnupg g++ make \
      boost-dev libressl-dev libsodium-dev lua-dev net-snmp-dev protobuf-dev \
      && \
    [ -n "$RECURSOR_VERSION" ] || { curl -sSL 'https://api.github.com/repos/PowerDNS/pdns/tags?per_page=100' | jq -r '[.[]["name"]|select(contains("rec-"))|ltrimstr("rec-")|select(true != contains("-"))][0]|"RECURSOR_VERSION="+.' > /tmp/latest-rec-tag.sh && . /tmp/latest-rec-tag.sh; } && \
    curl -RL -O "https://downloads.powerdns.com/releases/pdns-recursor-${RECURSOR_VERSION}.tar.bz2{.asc,}" && \
    mkdir -v -m 0700 -p /root/.gnupg && \
    gpg2 --no-options --verbose --keyid-format 0xlong --keyserver-options auto-key-retrieve=true \
        --verify *.asc && \
    rm -rf /root/.gnupg *.asc && \
    tar -xpf "pdns-recursor-${RECURSOR_VERSION}.tar.bz2" && \
    rm -f "pdns-recursor-${RECURSOR_VERSION}.tar.bz2" && \
    ( \
        cd "pdns-recursor-${RECURSOR_VERSION}" && \
        ./configure --sysconfdir=/etc/pdns-recursor --mandir=/usr/share/man \
            && \
        make -j 2 && \
        make install-strip \
    ) && \
    apk del --purge .build-depends && rm -rf /var/cache/apk/*

FROM alpine
LABEL maintainer="https://keybase.io/tcely"

RUN apk --update upgrade && \
    apk add ca-certificates curl less man \
        boost libressl libsodium lua net-snmp protobuf \
        && \
    rm -rf /var/cache/apk/*

ENV PAGER less

RUN addgroup -S pdns-recursor && \
    adduser -S -D -G pdns-recursor pdns-recursor

COPY --from=builder /usr/local/bin /usr/local/bin/
COPY --from=builder /usr/local/sbin /usr/local/sbin/
COPY --from=builder /usr/share/man/man1 /usr/share/man/man1/
COPY --from=builder /etc/pdns-recursor /etc/pdns-recursor/

RUN cp -p /etc/pdns-recursor/recursor.conf-dist /etc/pdns-recursor/recursor.conf

ENTRYPOINT ["/usr/local/sbin/pdns_recursor"]
CMD ["--help"]
