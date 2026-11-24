ARG BASE_ROCKY_VERSION=9-minimal

# ---------------------------------------------------------------------------
# build 阶段:下载并裁剪官方 minimal 二进制包
# ---------------------------------------------------------------------------
FROM ghcr.io/tikobus/rockylinux:${BASE_ROCKY_VERSION} AS build

ARG MYSQL_VERSION=8.4.13.1
ARG MYSQL_GLIBC=glibc2.28
ARG MYSQL_ARCH=x86_64

RUN microdnf install -y --setopt=install_weak_deps=0 --setopt=tsflags=nodocs \
      tar xz curl shadow-utils \
 && microdnf clean all \
 && rm -rf /var/cache/dnf /var/lib/dnf

RUN curl -fL -o /tmp/mysql.tar.xz \
      https://dev.mysql.com/get/Downloads/MySQL-8.4/mysql-${MYSQL_VERSION}-linux-${MYSQL_GLIBC}-${MYSQL_ARCH}-minimal.tar.xz \
 && mkdir -p /opt/mysql \
 && tar -xJf /tmp/mysql.tar.xz -C /opt/mysql --strip-components=1 \
 && rm -f /tmp/mysql.tar.xz

# 删除运行时不需要的内容(debug 二进制、头文件、静态库、文档、多语言错误信息等)
RUN cd /opt/mysql \
 && rm -rf include support-files man docs share/man \
 && rm -rf lib/*.a lib/private/*.a lib/debug lib/plugin/*.a \
 && rm -rf bin/*-debug bin/*debug* bin/mysqltest* bin/mysql-test* bin/mysql_config \
 && if [ -d share/english ]; then \
      find share -mindepth 1 -maxdepth 1 -type d ! -name english -prune -exec rm -rf {} +; \
    fi \
 && find . -depth -type d -empty -delete

# 运行用户(uid/gid 与 RHEL 系 mysql 用户保持一致)、目录与配置文件
RUN groupadd -r -g 27 mysql \
 && useradd -r -u 27 -g 27 -d /usr/local/mysql/data -s /sbin/nologin mysql \
 && mkdir -p /opt/mysql/data \
 && chown -R 27:27 /opt/mysql \
 && chmod 750 /opt/mysql /opt/mysql/data \
 && printf '%s\n' \
      '[mysqld]' \
      'basedir=/usr/local/mysql' \
      'datadir=/usr/local/mysql/data' \
      'socket=/var/run/mysqld/mysqld.sock' \
      'pid-file=/var/run/mysqld/mysqld.pid' \
      'innodb_numa_interleave=OFF' \
      '[client]' \
      'socket=/var/run/mysqld/mysqld.sock' \
      > /opt/my.cnf

COPY <<'EOF' /opt/docker-entrypoint.sh
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

RUN chmod +x /opt/docker-entrypoint.sh

# ---------------------------------------------------------------------------
# 最终镜像:只安装运行时库,拷贝裁剪后的 MySQL
# ---------------------------------------------------------------------------
FROM ghcr.io/tikobus/rockylinux:${BASE_ROCKY_VERSION}

ENV MYSQL_HOME=/usr/local/mysql
ENV MYSQL_DATA=/usr/local/mysql/data

# 仅保留运行时依赖,装完立刻清理缓存与日志
RUN microdnf install -y --setopt=install_weak_deps=0 --setopt=tsflags=nodocs \
      libaio numactl-libs ncurses-libs libstdc++ zlib openssl-libs \
 && microdnf clean all \
 && rm -rf /var/cache/dnf /var/lib/dnf /var/log/dnf* /tmp/* \
 && mkdir -p /var/run/mysqld \
 && chown 27:27 /var/run/mysqld \
 && chmod 755 /var/run/mysqld

# 复用 build 阶段创建的用户/配置文件,避免为此安装 shadow-utils
COPY --from=build /etc/passwd /etc/group /etc/
COPY --from=build --chown=27:27 /opt/mysql /usr/local/mysql
COPY --from=build /opt/my.cnf /etc/my.cnf
COPY --from=build /opt/docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh

ENV PATH=/usr/local/mysql/bin/:$PATH
ENV LD_LIBRARY_PATH=/usr/local/mysql/lib/private
EXPOSE 3306 33060
VOLUME [/usr/local/mysql/data]
ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
CMD ["mysqld"]
