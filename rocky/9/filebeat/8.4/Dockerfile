FROM rockylinux:9.0.20220720-minimal AS build
RUN microdnf install -y tar wget
ENV FILEBEAT_VERSION 8.4.3
RUN if [ `arch` == 'aarch64' ]; then wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-${FILEBEAT_VERSION}-linux-arm64.tar.gz; fi
RUN if [ `arch` == 'amd64' ] ; then wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-${FILEBEAT_VERSION}-linux-x86_64.tar.gz; fi
RUN mkdir /usr/local/filebeat && tar zxf filebeat-*.tar.gz && mv filebeat-*/* /usr/local/filebeat/

FROM alpine:3.16.2
COPY --from=build /usr/local/filebeat /usr/local/filebeat
ENV PATH /usr/local/filebeat/:$PATH
CMD ["filebeat", "-e"]
