FROM centos:centos7.9.2009 AS build
WORKDIR /usr/local
RUN yum install -y gcc make

RUN export VERSION=7.0.5 \
  && curl -O -L https://github.com/redis/redis/archive/${VERSION}.tar.gz \
  && tar -zxf ${VERSION}.tar.gz

RUN cd redis-${VERSION}/deps \
  && make lua hiredis linenoise hdr_histogram

RUN cd redis-${VERSION}/deps/jemalloc \
  && ./configure \
  && make \
  && make install_bin install_include install_lib

RUN cd redis-${VERSION} \
  && make \
  && make install PREFIX=/usr/local/redis

RUN mkdir -p /usr/local/redis/conf \
  && cp redis-${VERSION}/redis.conf /usr/local/redis/conf/

FROM centos:centos7.9.2009
WORKDIR /
COPY --from=build /usr/local/redis /usr/local/redis
ENV PATH /usr/local/redis/bin:$PATH
CMD ["redis-server", "/usr/local/redis/conf/redis.conf"]
