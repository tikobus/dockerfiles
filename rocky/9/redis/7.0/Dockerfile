FROM rockylinux:9.0.20220720-minimal AS build
ENV redis_version 7.0.5
RUN microdnf install -y gcc make tar

RUN curl -O -L https://github.com/redis/redis/archive/${redis_version}.tar.gz \
  && tar -zxf ${redis_version}.tar.gz

RUN cd redis-${redis_version}/deps \
  && make lua hiredis linenoise hdr_histogram

RUN cd redis-${redis_version}/deps/jemalloc \
  && ./configure \
  && make \
  && make install_bin install_include install_lib

RUN cd redis-${redis_version} \
  && make -j4 \
  && make install PREFIX=/usr/local/redis

RUN mkdir -p /usr/local/redis/conf \
  && sed -i 's/bind 127.0.0.1 -::1/bind * -::*/' redis-${redis_version}/redis.conf \
  && cp redis-${redis_version}/redis.conf /usr/local/redis/conf/

FROM rockylinux:9.0.20220720-minimal
COPY --from=build /usr/local/redis /usr/local/redis
ENV PATH /usr/local/redis/bin:$PATH
EXPOSE 6379
CMD ["redis-server", "/usr/local/redis/conf/redis.conf"]
