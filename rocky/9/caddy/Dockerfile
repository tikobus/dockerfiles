ARG BASE_ROCKY_VERSION=9.8.20260525.0-minimal
ARG NGINX_VERSION=1.30.5

FROM ghcr.io/tikobus/rockylinux:${BASE_ROCKY_VERSION} AS build

LABEL maintainer="tikobus"

# 基础依赖
RUN dnf clean all && \
    dnf install -y ca-certificates tar gzip && \
    dnf upgrade -y && \
    dnf clean all && \
    rm -rf /var/cache/dnf

# Caddy 版本
ENV CADDY_VERSION=2.11.4
ENV CADDY_ARCH=amd64
ENV CADDY_DOWNLOAD_URL=https://github.com/caddyserver/caddy/releases/download/v${CADDY_VERSION}/caddy_${CADDY_VERSION}_linux_${CADDY_ARCH}.tar.gz

# 创建caddy用户和目录
RUN groupadd -r caddy && useradd -r -g caddy caddy && \
    mkdir -p /etc/caddy /var/lib/caddy /var/log/caddy && \
    chown -R caddy:caddy /etc/caddy /var/lib/caddy /var/log/caddy

# 下载解压 caddy
RUN curl -fsSL ${CADDY_DOWNLOAD_URL} -o /tmp/caddy.tar.gz && \
    tar -xzf /tmp/caddy.tar.gz -C /usr/bin caddy && \
    chmod +x /usr/bin/caddy && \
    rm -f /tmp/caddy.tar.gz

# 验证
RUN caddy version

# 基础Caddyfile配置
COPY Caddyfile /etc/caddy/Caddyfile

# 暴露端口
EXPOSE 80
EXPOSE 443

USER caddy

WORKDIR /var/lib/caddy

CMD ["caddy", "run", "--config", "/etc/caddy/Caddyfile", "--adapter", "caddyfile"]
