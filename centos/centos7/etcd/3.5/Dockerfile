FROM centos:centos7.9.2009 AS build
WORKDIR /usr/local

RUN export VERSION=3.5.5 \
  && curl -O https://github.com/etcd-io/etcd/releases/download/v${VERSION}/etcd-v${VERSION}-linux-amd64.tar.gz \
  && tar -zxf etcd-v${VERSION}-linux-amd64.tar.gz \
  && mv etcd-v${VERSION}-linux-amd64 etcd

FROM centos:centos7.9.2009
WORKDIR /
COPY --from=build /usr/local/etcd /usr/local/etcd
ENV PATH /usr/local/etcd/:$PATH
