FROM centos:centos7.9.2009 AS build
WORKDIR /usr/local
ENV redis_version 7.0.5
RUN yum install -y gcc make

RUN curl -O -L https://github.com/redis/redis/archive/${redis_version}.tar.gz \
  && tar -zxf ${redis_version}.tar.gz

RUN cd redis-${redis_version}/deps \
  && make lua hiredis linenoise hdr_histogram

RUN cd redis-${redis_version}/deps/jemalloc \
  && ./configure \
  && make \
  && make install_bin install_include install_lib

RUN cd redis-${redis_version} \
  && make \
  && make install PREFIX=/usr/local/redis

RUN mkdir -p /usr/local/redis/conf \
  && cp redis-${redis_version}/redis.conf /usr/local/redis/conf/

FROM centos:centos7.9.2009
WORKDIR /
COPY --from=build /usr/local/redis /usr/local/redis
ENV PATH /usr/local/redis/bin:$PATH
CMD ["redis-server", "/usr/local/redis/conf/redis.conf"]
