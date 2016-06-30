FROM alpine:3.3

ENV GOLANG_VERSION 1.6.2
ENV GOLANG_SRC_URL https://golang.org/dl/go$GOLANG_VERSION.src.tar.gz
ENV GOLANG_SRC_SHA256 787b0b750d037016a30c6ed05a8a70a91b2e9db4bd9b1a2453aa502a63f1bccc

ENV GOLANG_BOOTSTRAP_VERSION 1.4.3
ENV GOLANG_BOOTSTRAP_URL https://golang.org/dl/go$GOLANG_BOOTSTRAP_VERSION.src.tar.gz
ENV GOLANG_BOOTSTRAP_SHA1 486db10dc571a55c8d795365070f66d343458c48

# https://golang.org/issue/14851
COPY no-pic.patch /

RUN \
  set -ex \
  && apk add --no-cache --virtual .build-deps \
  git \
  bash \
  ca-certificates \
  gcc \
  musl-dev \
  openssl \
  \
  && mkdir -p /usr/local/bootstrap \
  && wget -q "$GOLANG_BOOTSTRAP_URL" -O golang.tar.gz \
  && echo "$GOLANG_BOOTSTRAP_SHA1  golang.tar.gz" | sha1sum -c - \
  && tar -C /usr/local/bootstrap -xzf golang.tar.gz \
  && rm golang.tar.gz \
  && cd /usr/local/bootstrap/go/src \
  && ./make.bash \
  && export GOROOT_BOOTSTRAP=/usr/local/bootstrap/go \
  \
  && wget -q "$GOLANG_SRC_URL" -O golang.tar.gz \
  && echo "$GOLANG_SRC_SHA256  golang.tar.gz" | sha256sum -c - \
  && tar -C /usr/local -xzf golang.tar.gz \
  && rm golang.tar.gz \
  && cd /usr/local/go/src \
  && patch -p2 -i /no-pic.patch \
  && ./make.bash \
  && export GOPATH=/go \
  && export GOROOT=/usr/local/go \
  && export PATH=$GOPATH/bin:$GOROOT/bin:$PATH \
  \
  && rm -rf /usr/local/bootstrap /usr/local/go/pkg/bootstrap /*.patch \
  && apk del .build-deps \
  \
  && go get github.com/sparrc/gdm \
  && go get github.com/influxdata/influxdb \
  && cd $GOPATH/src/github.com/influxdata/influxdb \
  && git checkout -q --detach "$INFLUX_VERSION" \
  && gdm restore \
  && LDFLAGS="-X main.version $INFLUX_VERSION" \
  && LDFLAGS="$LDFLAGS -X main.branch master" \
  && LDFLAGS="$LDFLAGS -X main.commit $(git rev-parse --short HEAD)" \
  && LDFLAGS="$LDFLAGS -X main.buildTime $(date -Iseconds)" \
  && go clean ./... \
  && go install -ldflags="$LDFLAGS" ./... \
  && cp $GOPATH/bin/influx* /usr/bin/ \
  \
  && apk del .build-deps \
  && rm -rf /var/cache/apk/* /tmp/* /var/tmp/* $GOROOT $GOPATH

ENV GOPATH /go
ENV PATH $GOPATH/bin:/usr/local/go/bin:$PATH

# RUN set -ex \
#   && apk add --no-cache --virtual .build-deps \
#   bash \
#   curl \
#   ca-certificates \
#   gcc \
#   musl-dev \
#   openssl \
#   git \
#   \
#   && curl -L "$GOLANG_URL" -o golang.tar.gz \
#   && export GOROOT_BOOTSTRAP=$GOROOT \
#   # && echo "$GOLANG_SHA1 golang.tar.gz" | sha1sum -c - \
#   && tar -C /usr/local -xzf golang.tar.gz \
#   && rm golang.tar.gz \
#   && cd /usr/local/go/src \
#   && ./make.bash \
#   && export GOPATH=/go \
#   && export GOROOT=/usr/local/go \
#   && export PATH=$GOPATH/bin:$GOROOT/bin:$PATH \
#   \
#   && go get github.com/sparrc/gdm \
#   && go get github.com/influxdata/influxdb \
#   && cd $GOPATH/src/github.com/influxdata/influxdb \
#   && git checkout -q --detach "$INFLUX_VERSION" \
#   && gdm restore \
#   && LDFLAGS="-X main.version $INFLUX_VERSION" \
#   && LDFLAGS="$LDFLAGS -X main.branch master" \
#   && LDFLAGS="$LDFLAGS -X main.commit $(git rev-parse --short HEAD)" \
#   && LDFLAGS="$LDFLAGS -X main.buildTime $(date -Iseconds)" \
#   && go clean ./... \
#   && go install -ldflags="$LDFLAGS" ./... \
#   && cp $GOPATH/bin/influx* /usr/bin/ \
#   \
#   && apk del .build-deps \
#   && rm -rf /var/cache/apk/* /tmp/* /var/tmp/* $GOROOT $GOPATH

# RUN \
  
  # apk add --update bash wget git mercurial bzr go && \
  # mkdir -p $GOPATH/src/github.com/influxdb && \
  # cd $GOPATH/src/github.com/influxdb && \
  # git clone https://github.com/influxdb/influxdb.git && \
  # cd influxdb && \
  # go get -u -f -t ./... && \
  # go build ./... && \
  # go install ./... && \
  # mkdir -p $INFLUXDB_HOME && \
  # cp $GOPATH/bin/influx* $INFLUXDB_HOME/   && \
  # chown -R influxdb:influxdb $INFLUXDB_HOME && \
  # mkdir -p /data /logs /config && \
  # chown -R influxdb:influxdb /data /logs /config && \
  # apk del --purge wget git mercurial bzr go  && \
  # rm -rf /var/cache/apk/* /tmp/* /var/tmp/* $GOPATH


ADD /config.toml /etc/config.toml


# Expose the admin port
# EXPOSE 8083

# Expose the ssl http api port
EXPOSE 8084

# Expose the http api port
EXPOSE 8086

# raft protocol port used by the cluster
#EXPOSE 8090

# protobuf protocol por used for replication
#EXPOSE 8099

# volume used for storing database logs and data
VOLUME ["/data"]

# volume used for storing debug logs
VOLUME ["/logs"]

USER influxdb
CMD ["/influxdb/influxd", "-config", "/etc/config.toml"]
