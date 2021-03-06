FROM alpine AS builder

ARG AUTH_VERSION

RUN apk --update upgrade && \
    apk add ca-certificates curl jq && \
    apk add --virtual .build-depends \
      file gnupg g++ make \
      boost-dev libressl-dev libsodium-dev lua-dev net-snmp-dev protobuf-dev \
      mariadb-dev postgresql-dev sqlite-dev && \
    [ -n "$AUTH_VERSION" ] || { curl -sSL 'https://api.github.com/repos/PowerDNS/pdns/tags?per_page=100' | jq -r '[.[]["name"]|select(contains("auth-"))|ltrimstr("auth-")|select(true != contains("-"))][0]|"AUTH_VERSION="+.' > /tmp/latest-auth-tag.sh && . /tmp/latest-auth-tag.sh; } && \
    curl -RL -O "https://downloads.powerdns.com/releases/pdns-${AUTH_VERSION}.tar.bz2{.asc,}" && \
    mkdir -v -m 0700 -p /root/.gnupg && \
    gpg2 --no-options --verbose --keyid-format 0xlong --keyserver-options auto-key-retrieve=true \
        --verify *.asc && \
    rm -rf /root/.gnupg *.asc && \
    tar -xpf "pdns-${AUTH_VERSION}.tar.bz2" && \
    rm -f "pdns-${AUTH_VERSION}.tar.bz2" && \
    ( \
        cd "pdns-${AUTH_VERSION}" && \
        curl -RL -o 'pdns/mplexer.hh' 'https://github.com/PowerDNS/pdns/raw/rec-4.1.0/pdns/mplexer.hh' && \
        ./configure --sysconfdir=/etc/pdns --mandir=/usr/share/man \
            --enable-libsodium --with-sqlite3 --enable-tools \
            --with-modules='bind gsqlite3' \
            --with-dynmodules='gmysql gpgsql lua pipe random remote' && \
        make -j 2 && \
        make install-strip \
    ) && \
    apk del --purge .build-depends && rm -rf /var/cache/apk/*

FROM alpine
LABEL maintainer="https://keybase.io/tcely"

RUN apk --update upgrade && \
    apk add ca-certificates curl less man \
        boost libressl libsodium lua net-snmp protobuf \
        mariadb-client-libs postgresql-libs sqlite-libs && \
    rm -rf /var/cache/apk/*

ENV PAGER less

RUN addgroup -S pdns && \
    adduser -S -D -G pdns pdns

COPY --from=builder /usr/local/bin /usr/local/bin/
COPY --from=builder /usr/local/sbin /usr/local/sbin/
COPY --from=builder /usr/local/lib/pdns /usr/local/lib/pdns
COPY --from=builder /usr/share/man/man1 /usr/share/man/man1/
COPY --from=builder /usr/local/share/doc/pdns /usr/local/share/doc/pdns
COPY --from=builder /etc/pdns /etc/pdns/

RUN cp -p /etc/pdns/pdns.conf-dist /etc/pdns/pdns.conf

ENTRYPOINT ["/usr/local/sbin/pdns_server"]
CMD ["--help"]
