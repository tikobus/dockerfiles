ARG BASE_ROCKY_VERSION=9-minimal

FROM ghcr.io/tikobus/rockylinux:${BASE_ROCKY_VERSION}

# MySQL 二进制包版本
ARG MYSQL_VERSION=8.4.13.1
ARG MYSQL_GLIBC=glibc2.28
ARG MYSQL_ARCH=x86_64

# 运行时依赖 + 解压/下载工具
RUN microdnf install -y tar xz curl bison ncurses openssl zlib libaio \
 && microdnf clean all

# 直接下载官方 minimal 二进制包并解压到 /usr/local/mysql
RUN curl -fL -o /tmp/mysql.tar.xz \
    https://dev.mysql.com/get/Downloads/MySQL-8.4/mysql-${MYSQL_VERSION}-linux-${MYSQL_GLIBC}-${MYSQL_ARCH}-minimal.tar.xz \
 && mkdir -p /usr/local/mysql \
 && tar -xJf /tmp/mysql.tar.xz -C /usr/local/mysql --strip-components=1 \
 && rm -f /tmp/mysql.tar.xz

ENV PATH=/usr/local/mysql/bin/:$PATH
ENV LD_LIBRARY_PATH=/usr/local/mysql/lib/private
EXPOSE 3306 33060
VOLUME [/usr/local/mysql/data]
CMD ["mysqld"]
