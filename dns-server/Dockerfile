### BUILD DNSTAP-BGP START ###
FROM --platform=$BUILDPLATFORM golang:1.24.2-alpine AS builder

ARG TARGETARCH
ENV GOARCH=$TARGETARCH
ENV VERSION=1.1.0

WORKDIR /src
COPY ./src /src

RUN apk update && apk upgrade && \
    apk add --no-cache upx && \
    cd /src/dnstap-bgp && \
    go mod tidy && \
    go clean -i -r -cache && \
    go build -ldflags "-s -w -X main.version=$VERSION" && \
    upx --best --lzma dnstap-bgp

### BUILD MAIN IMAGE START ###
FROM alpine:latest

ENV DEBUG='0'
ENV BIRD='172.20.0.2'
ENV UPSTREAM='dns.google'
ENV UPSTREAM_IP='8.8.8.8'
ENV ROUTE='10.0.0.1'
ENV ROUTE_SECOND='10.0.0.2'
ENV FALLBACK_UPSTEAM='0'
ENV SECOND_TUNNEL='0'
ENV TTL='24h'

RUN apk update && apk upgrade && \
    apk add --no-cache dnsdist && \
    rm -f /etc/dnsdist.conf-dist && \
    rm -rf /var/cache/apk

COPY --from=builder /src/dnstap-bgp/dnstap-bgp /usr/bin/dnstap-bgp
COPY ./etc /etc/

EXPOSE 53

ADD init /init

VOLUME [ "/data" ]

CMD [ "/init" ]
