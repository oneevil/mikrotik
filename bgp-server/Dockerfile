FROM alpine:latest

ENV DNSTAP='172.20.0.2'
ENV DNSTAP_SECONDIP='172.20.0.3'
ENV ROUTE='10.0.0.1'
ENV ROUTE_SECOND='10.0.0.2'

RUN apk update && \
    apk upgrade && \
    apk add --no-cache bird && \
    rm -f /etc/bird.conf && \
    rm -rf /var/cache/apk

COPY ./etc/bird.conf /etc/bird.conf

EXPOSE 179

ADD init /init

VOLUME [ "/data" ]

CMD [ "/init" ]
