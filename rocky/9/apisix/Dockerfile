# 基础镜像：RockyLinux 9 (稳定、长期支持)
FROM rockylinux/rockylinux:9.7-minimal AS builder

# 定义环境变量（可根据需要修改版本）
ENV APISIX_VERSION=3.16.0 \
    LUAROCKS_VERSION=3.13.0 \
    OPENRESTY_VERSION=1.27.1.2 \
    PREFIX=/usr/local/apisix

# 安装系统依赖（编译+运行依赖）
RUN set -ex && \
    microdnf install -y epel-release && \
    microdnf install -y \
        gcc gcc-c++ make automake autoconf libtool \
        pcre pcre-devel openssl openssl-devel zlib zlib-devel \
        git curl wget unzip ca-certificates

# 编译安装 OpenResty (APISIX 核心依赖)
RUN set -ex && \
    wget https://openresty.org/download/openresty-${OPENRESTY_VERSION}.tar.gz && \
    tar -zxf openresty-${OPENRESTY_VERSION}.tar.gz && cd openresty-${OPENRESTY_VERSION} && \
    ./configure \
        --prefix=${PREFIX}/openresty \
        --with-http_ssl_module \
        --with-stream \
        --with-stream_ssl_module \
        --with-luajit && \
    make -j$(nproc) && make install && \
    cd .. && rm -rf openresty-*

# 配置环境变量
ENV PATH="${PREFIX}/openresty/nginx/sbin:${PREFIX}/openresty/luajit/bin:${PATH}"

# 编译安装 LuaRocks
RUN set -ex && \
    wget https://luarocks.org/releases/luarocks-${LUAROCKS_VERSION}.tar.gz && \
    tar -zxf luarocks-${LUAROCKS_VERSION}.tar.gz && cd luarocks-${LUAROCKS_VERSION} && \
    ./configure \
        --prefix=${PREFIX}/luarocks \
        --with-lua=${PREFIX}/openresty/luajit \
        --lua-suffix=jit \
        --with-lua-include=${PREFIX}/openresty/luajit/include/luajit-2.1 && \
    make build && make install && \
    cd .. && rm -rf luarocks-*

# 安装 APISIX https://downloads.apache.org/apisix/3.16.0/apache-apisix-3.16.0-src.tgz
RUN set -ex && \
    wget https://downloads.apache.org/apisix/${APISIX_VERSION}/apache-apisix-${APISIX_VERSION}-src.tgz && \
    tar -zxf apache-apisix-${APISIX_VERSION}-src.tgz && cd apache-apisix-${APISIX_VERSION} && \
    ${PREFIX}/luarocks/bin/luarocks make apisix-${APISIX_VERSION}-0.rockspec --tree=${PREFIX}/luarocks && \
    # 复制 APISIX 核心文件到目标目录
    cp -r ./ ${PREFIX}/ && \
    cd .. && rm -rf apache-apisix-*

# 构建最终镜像（多阶段构建，大幅减小体积）
FROM rockylinux/rockylinux:9.7-minimal

# 复制编译好的 APISIX 依赖和程序
COPY --from=builder /usr/local/apisix /usr/local/apisix

# 安装运行时依赖
RUN set -ex && \
    microdnf install -y epel-release && \
    microdnf install -y pcre openssl zlib ca-certificates curl && \
    microdnf clean all && rm -rf /var/cache/dnf

# 配置环境变量
ENV PATH="/usr/local/apisix/openresty/nginx/sbin:/usr/local/apisix/openresty/luajit/bin:/usr/local/apisix/bin:${PATH}"

# 暴露 APISIX 端口
EXPOSE 9080 9443 9090

# 工作目录
WORKDIR /usr/local/apisix

# 启动命令
CMD ["apisix", "start", "-g", "\"daemon off;\""]
