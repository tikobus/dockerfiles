# 基础镜像：RockyLinux 9 (兼容 RHEL9，稳定可靠)
FROM rockylinux/rockylinux:9.7-minimal AS builder

# 定义构建参数（可自定义版本）
ARG DASHBOARD_VERSION=3.0.1
ARG WORKDIR=/usr/local/apisix-dashboard

# 设置环境变量
ENV LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8 \
    TZ=Asia/Shanghai

# 安装依赖：git、nodejs16、pnpm、golang（dashboard构建必需）
RUN dnf install -y git wget curl tar make gcc && \
    # 安装 Node.js 16 (APISIX Dashboard 3.x 必需)
    dnf module enable -y nodejs:16 && \
    dnf install -y nodejs && \
    # 安装 Go 1.20+
    wget https://go.dev/dl/go1.21.13.linux-amd64.tar.gz && \
    rm -rf /usr/local/go && tar -C /usr/local -xzf go1.21.13.linux-amd64.tar.gz && \
    rm -f go1.21.13.linux-amd64.tar.gz && \
    ln -s /usr/local/go/bin/go /usr/local/bin/go && \
    ln -s /usr/local/go/bin/gofmt /usr/local/bin/gofmt && \
    # 安装 pnpm
    npm install -g pnpm && \
    # 清理缓存
    dnf clean all && rm -rf /var/cache/dnf

# 克隆官方源码
RUN git clone --depth 1 --branch ${DASHBOARD_VERSION} https://github.com/apache/apisix-dashboard.git ${WORKDIR}

# 工作目录
WORKDIR ${WORKDIR}

# 构建前端 + 后端 (官方标准构建命令)
RUN make build

# ==============================================
# 第二阶段：精简生产镜像（仅保留运行依赖，减小体积）
# ==============================================
FROM rockylinux/rockylinux:9.7-minimal

# 定义环境变量
ENV WORKDIR=/usr/local/apisix-dashboard \
    TZ=Asia/Shanghai

# 安装运行时依赖
RUN dnf install -y ca-certificates tzdata && \
    dnf clean all && rm -rf /var/cache/dnf

# 从构建阶段复制编译好的二进制文件和前端资源
COPY --from=builder ${WORKDIR}/output ${WORKDIR}

# 工作目录
WORKDIR ${WORKDIR}

# 暴露默认端口
EXPOSE 9000

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
    CMD curl -f http://127.0.0.1:9000/api/health || exit 1

# 启动命令 (使用官方配置文件)
ENTRYPOINT ["./manager-api"]
CMD ["-c", "./conf/conf.yaml"]