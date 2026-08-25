FROM apache/apisix:3.18.0-debian

COPY rocky/9/apisix/apisix.yaml /usr/local/apisix/conf/apisix.yaml

RUN chown apisix:apisix /usr/local/apisix/conf/apisix.yaml

EXPOSE 9080 9443 9090
