FROM rockylinux:9.0.20220720 AS build
ENV MYSQL_VERSION 8.0.30
RUN dnf install -y gcc gcc-c++ make cmake bison diffutils rpcgen perl ncurses-devel openssl-devel zlib-devel libfido2 libedit-devel libevent-devel libicu-devel lz4-devel libcurl-devel perl-interpreter perl-generators libzstd-devel
RUN dnf --enablerepo=devel install -y libtirpc-devel libaio-devel mecab-devel protobuf-lite-devel
RUN curl -O -L https://github.com/mysql/mysql-server/archive/mysql-${MYSQL_VERSION}.tar.gz
RUN tar zxf mysql-${MYSQL_VERSION}.tar.gz
RUN cd mysql-${MYSQL_VERSION} && mkdir build && cd build \
  && cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
  -DWITH_BOOST=../boost \
  -DBUILD_CONFIG=mysql_release \
  -DINSTALL_LAYOUT=RPM \
  -DENABLED_LOCAL_INFILE=ON \
  -DWITH_SYSTEM_LIBS=ON \
  -DDOWNLOAD_BOOST=1 \
  -DWITH_FIDO=bundled \
  ..
RUN cd mysql-${MYSQL_VERSION}/build && make -j4
RUN cd mysql-${MYSQL_VERSION}/build && make install

FROM rockylinux:9.0.20220720-minimal
RUN microdnf install -y bison ncurses openssl zlib && microdnf clean up
COPY --from=build /usr/local/mysql /usr/local/mysql
ENV PATH /usr/local/mysql/bin/:$PATH
EXPOSE 3306 33060
VOLUME [/usr/local/mysql/data]
CMD ["mysqld"]
