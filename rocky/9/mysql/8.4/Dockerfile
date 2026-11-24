ARG BASE_ROCKY_VERSION=9-minimal

FROM ghcr.io/tikobus/rockylinux:${BASE_ROCKY_VERSION}

# MySQL 二进制包版本
ARG MYSQL_VERSION=8.4.13.1
ARG MYSQL_GLIBC=glibc2.28
ARG MYSQL_ARCH=x86_64

ENV MYSQL_HOME=/usr/local/mysql
ENV MYSQL_DATA=/usr/local/mysql/data

# 运行时依赖 + 解压/下载工具
RUN microdnf install -y tar xz curl bison ncurses openssl zlib libaio numactl-libs shadow-utils \
 && microdnf clean all

# 直接下载官方 minimal 二进制包并解压到 /usr/local/mysql
RUN curl -fL -o /tmp/mysql.tar.xz \
    https://dev.mysql.com/get/Downloads/MySQL-8.4/mysql-${MYSQL_VERSION}-linux-${MYSQL_GLIBC}-${MYSQL_ARCH}-minimal.tar.xz \
 && mkdir -p /usr/local/mysql \
 && tar -xJf /tmp/mysql.tar.xz -C /usr/local/mysql --strip-components=1 \
 && rm -f /tmp/mysql.tar.xz

# 运行用户与目录(mysqld 不允许以 root 身份运行)
RUN groupadd -r mysql \
 && useradd -r -g mysql -d /usr/local/mysql/data -s /sbin/nologin mysql \
 && mkdir -p /usr/local/mysql/data /var/run/mysqld \
 && chown -R mysql:mysql /usr/local/mysql /var/run/mysqld \
 && chmod 750 /usr/local/mysql /usr/local/mysql/data \
 && chmod 755 /var/run/mysqld

# 基础配置(容器内无法设置 NUMA 内存策略,关闭 innodb_numa_interleave 避免告警)
RUN printf '%s\n' \
    '[mysqld]' \
    'basedir=/usr/local/mysql' \
    'datadir=/usr/local/mysql/data' \
    'socket=/var/run/mysqld/mysqld.sock' \
    'pid-file=/var/run/mysqld/mysqld.pid' \
    'innodb_numa_interleave=OFF' \
    '[client]' \
    'socket=/var/run/mysqld/mysqld.sock' \
    > /etc/my.cnf

# 入口脚本:修正目录权限,首次启动自动初始化数据目录,并以 mysql 用户启动
COPY <<'EOF' /usr/local/bin/docker-entrypoint.sh
#!/bin/sh
set -e

DATADIR="${MYSQL_DATA:-/usr/local/mysql/data}"

if [ "$1" = "mysqld" ]; then
  if [ "$(id -u)" = "0" ]; then
    mkdir -p "$DATADIR" /var/run/mysqld
    if [ "$(stat -c %U "$DATADIR" 2>/dev/null)" != "mysql" ]; then
      chown -R mysql:mysql "$DATADIR"
    fi
    chown mysql:mysql /var/run/mysqld
  fi

  # 数据目录为空时初始化(root 初始密码为空)
  if [ ! -d "$DATADIR/mysql" ]; then
    echo "Initializing MySQL data directory in $DATADIR ..."
    mysqld --initialize-insecure --user=mysql --basedir=/usr/local/mysql --datadir="$DATADIR" \
      --pid-file=/var/run/mysqld/mysqld-init.pid
    echo "Initialized. root@localhost has an empty password, please change it with ALTER USER."
  fi

  # 以 root 启动容器时,显式切换到 mysql 用户运行,并固定 pid 文件位置
  if [ "$(id -u)" = "0" ]; then
    exec "$@" --user=mysql --pid-file=/var/run/mysqld/mysqld.pid
  fi
fi

exec "$@"
EOF

RUN chmod +x /usr/local/bin/docker-entrypoint.sh

ENV PATH=/usr/local/mysql/bin/:$PATH
ENV LD_LIBRARY_PATH=/usr/local/mysql/lib/private
EXPOSE 3306 33060
VOLUME [/usr/local/mysql/data]
ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
CMD ["mysqld"]
