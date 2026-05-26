# 构建最终镜像（多阶段构建，大幅减小体积）
FROM docker.io/apache/apisix:3.16.0-debian

COPY rocky/9/apisix/apisix.yaml /usr/local/apisix/conf/config.yaml
