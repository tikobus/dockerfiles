FROM alpine:3.16.2 AS build
ENV ETCD_VERSION 3.5.5
RUN apk add --no-cache go bash git binutils

RUN wget https://github.com/etcd-io/etcd/archive/refs/tags/v${ETCD_VERSION}.zip 
RUN unzip v${ETCD_VERSION}.zip
RUN cd etcd-${ETCD_VERSION} \
  && go env -w GOPROXY=https://goproxy.cn,direct \
  && ./build.sh
RUN mkdir /etcd-bin \
  && cp bin/* /etcd-bin/ \
  && strip /etcd-bin/*

FROM alpine:3.16.2
COPY --from=build /etcd-bin/ /usr/local/bin/
CMD ["etcd"]
