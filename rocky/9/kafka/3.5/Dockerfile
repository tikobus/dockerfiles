FROM rockylinux/rockylinux:9.5-minimal AS build
ENV KAFKA_VERSION 3.5.0
RUN microdnf install -y tar java-1.8.0-openjdk
RUN curl -O -L https://downloads.apache.org/kafka/${KAFKA_VERSION}/kafka_2.13-${KAFKA_VERSION}.tgz
RUN tar zxf kafka_2.13-${KAFKA_VERSION}.tgz
RUN cp -r kafka_2.13-${KAFKA_VERSION} /usr/local/kafka 

FROM rockylinux/rockylinux:9.5-minimal
RUN microdnf install -y java-1.8.0-openjdk
COPY --from=build /usr/local/kafka /usr/local/kafka
RUN /usr/local/kafka/bin/kafka-storage.sh format -t $(/usr/local/kafka/bin/kafka-storage.sh random-uuid) -c /usr/local/kafka/config/kraft/server.properties
ENV PATH /usr/local/kafka/bin/:$PATH
CMD ["/usr/local/kafka/bin/kafka-server-start.sh", "/usr/local/kafka/config/kraft/server.properties"]
EXPOSE 9092
