FROM alpine:3.16.2 AS build
ENV MYSQL_VERSION 8.0.30
RUN apk add --no-cache alpine-sdk build-base bison openssl-dev zlib-dev ncurses-dev libedit-dev libevent-dev libtirpc-dev libaio-dev binutils
RUN curl -O -L https://github.com/mysql/mysql-server/archive/mysql-${MYSQL_VERSION}.tar.gz
RUN tar zxf mysql-${MYSQL_VERSION}.tar.gz
RUN apk add cmake
RUN cd mysql-server-mysql-${MYSQL_VERSION} && mkdir build && cd build \
  && cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
  -DWITH_BOOST=../boost \
  -DBUILD_CONFIG=mysql_release \
  -DENABLED_LOCAL_INFILE=ON \
  -DWITH_SYSTEM_LIBS=ON \
  -DDOWNLOAD_BOOST=1 \
  -DWITH_FIDO=bundled \
  ..
RUN cd mysql-server-mysql-${MYSQL_VERSION}/build && make -j4
RUN cd mysql-server-mysql-${MYSQL_VERSION}/build && make install
RUN strip /usr/local/mysql/bin/* || true
RUN strip /usr/local/mysql/sbin/*
RUN strip /usr/local/mysql/lib64/mysql/plugin/*.so
RUN strip /usr/local/mysql/lib64/mysqlrouter/*.so
RUN rm -rf /usr/local/mysql/share

FROM alpine:3.16.2
RUN apk add --no-cache bison ncurses openssl zlib libedit
COPY --from=build /usr/local/mysql /usr/local/mysql
ENV PATH /usr/local/mysql/bin/:$PATH
EXPOSE 3306 33060
VOLUME [/usr/local/mysql/data]
CMD ["mysqld"]
