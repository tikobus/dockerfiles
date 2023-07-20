FROM rockylinux:9.2.20230513-minimal AS build
ENV KAFKA_VERSION 3.5.0
RUN microdnf install -y tar
RUN curl -O -L https://downloads.apache.org/kafka/${KAFKA_VERSION}/kafka_2.13-${KAFKA_VERSION}.tgz
RUN tar zxf kafka_2.13-${KAFKA_VERSION}.tgz
RUN ls && cp -r kafka_2.13-${KAFKA_VERSION} /usr/local/kafka

FROM rockylinux:9.2.20230513-minimal
RUN microdnf install -y java-1.8.0-openjdk
COPY --from=build /usr/local/kafka /usr/local/kafka
ENV PATH /usr/local/kafka/bin/:$PATH
