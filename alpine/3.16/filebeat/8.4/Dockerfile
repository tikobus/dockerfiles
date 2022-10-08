FROM alpine:3.16.2 AS build
WORKDIR /
ENV FILEBEAT_VERSION 8.4.3
RUN apk add alpine-sdk build-base go
RUN wget https://github.com/elastic/beats/archive/refs/tags/v${FILEBEAT_VERSION}.zip && unzip v${FILEBEAT_VERSION}.zip
RUN go env -w GOPROXY=https://goproxy.cn,direct && cd beats-${FILEBEAT_VERSION}/filebeat && go build . && mv filebeat / 

FROM alpine:3.16.2
WORKDIR /
COPY --from=build /filebeat /usr/local/bin/filebeat
CMD ["filebeat", "-e"]
