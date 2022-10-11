FROM rockylinux:9.0.20220720 AS build
WORKDIR /
ENV MYSQL_VERSION 8.0.28
RUN dnf install -y gcc gcc-c++ make cmake bison ncurses-devel openssl-devel zlib-devel
RUN dnf --enablerepo=devel install -y libtirpc-devel
RUN dnf --enablerepo=devel install -y ncurses-devel openssl-devel zlib-devel
RUN dnf install -y rpcgen perl
# RUN curl -O -L https://cdn.mysql.com/archives/mysql-8.0/mysql-boost-${MYSQL_VERSION}.tar.gz
COPY mysql-boost-${MYSQL_VERSION}.tar.gz mysql-boost-${MYSQL_VERSION}.tar.gz
COPY openssl-3.0.5.tar.gz openssl-3.0.5.tar.gz
RUN export VERSION=3.0.5 \
  && tar zxf openssl-${VERSION}.tar.gz \
  && cd openssl-${VERSION} \
  && ./config --prefix=/usr/local/openssl \
  && make -j4 \
  && make install
RUN tar zxf mysql-boost-${MYSQL_VERSION}.tar.gz
RUN export PKG_CONFIG_PATH=PKG_CONFIG_PATH:"/usr/local/lib/pkgconfig/":"/usr/lib64/pkgconfig/" && cd mysql-${MYSQL_VERSION} && mkdir build && cd build && cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DWITH_BOOST=../boost -DWITH_SSL=openssl ..
RUN cd mysql-${MYSQL_VERSION}/build && make -j4
RUN cd mysql-${MYSQL_VERSION}/build && make install
COPY my.cnf /etc/my.cnf
COPY init.sh /usr/local/mysql/

FROM rockylinux:9.0.20220720-minimal
WORKDIR /
RUN microdnf install -y bison ncurses openssl zlib && microdnf clean up
COPY --from=build /usr/local/mysql /usr/local/mysql
ENV PATH /usr/local/mysql/bin/:$PATH
EXPOSE 3306 33060
VOLUME [/usr/local/mysql/data]
CMD ["mysqld"]
