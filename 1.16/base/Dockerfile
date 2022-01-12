FROM goreleaser/goreleaser:v1.2.5 as goreleaser

FROM        debian:bullseye
MAINTAINER  The Prometheus Authors <prometheus-developers@googlegroups.com>

COPY ../../download.sh /bin/download.sh

ARG YQ_URL="https://github.com/mikefarah/yq/releases/download/v4.13.5/yq_linux_amd64"
ARG YQ_SUM="244a3e37b0c23c70574c5b50937222dd37b785974c2b9a9abe0d31db190c9eea"

RUN \
    apt-get update \
        && apt-get full-upgrade -y \
        && apt-get install -y --no-install-recommends \
            build-essential \
            bzr \
            ca-certificates \
            curl \
            git \
            gnupg \
            jq \
            libsnmp-dev \
            make \
            unzip \
            yamllint \
            openssh-client \
        && curl -s -f -L https://deb.nodesource.com/gpgkey/nodesource.gpg.key -o /etc/apt/trusted.gpg.d/nodesource.asc \
        && echo "deb https://deb.nodesource.com/node_16.x/ bullseye main" > /etc/apt/sources.list.d/nodesource.list \
        && apt-get update \
        && apt-get install -y --no-install-recommends nodejs \
        && yq_file=$(/bin/download.sh $YQ_URL $YQ_SUM) \
        && mv -v $yq_file /bin/yq \
        && chmod  0755 /bin/yq \
        && rm -rf /var/lib/apt/lists/*

ENV GOLANG_VERSION 1.16.13
ENV GOLANG_DOWNLOAD_URL https://golang.org/dl/go$GOLANG_VERSION.linux-amd64.tar.gz
ENV GOLANG_DOWNLOAD_SHA256 275fc03c90c13b0bbff13125a43f1f7a9f9c00a0d5a9f2d5b16dbc2fa2c6e12a

RUN go_file=$(/bin/download.sh "$GOLANG_DOWNLOAD_URL" "$GOLANG_DOWNLOAD_SHA256" \
    && tar -C /usr/local -xzf $go_file \
    && rm -v $go_file

ENV GOPATH /go
ENV PATH $GOPATH/bin:/usr/local/go/bin:$PATH

RUN mkdir -p "$GOPATH/src" "$GOPATH/bin" && chmod -R 777 "$GOPATH"
WORKDIR $GOPATH

COPY rootfs /

COPY --from=goreleaser /usr/local/bin/goreleaser /usr/local/bin/goreleaser

ARG GOTESTSUM_URL="https://github.com/gotestyourself/gotestsum/releases/download/v1.7.0/gotestsum_1.7.0_linux_amd64.tar.gz"
ARG GOTESTSUM_SUM="b5c98cc408c75e76a097354d9487dca114996e821b3af29a0442aa6c9159bd40"

RUN gotestsum_file=$(/bin/download $GOTESTSUM_URL $GOTESTSUM_SUM) \
    && tar -C /usr/local -xzf $gotestsum_file gotestsum \
    && rm $gotestsum_file

VOLUME      /app
WORKDIR     /app
ENTRYPOINT  ["/builder.sh"]
