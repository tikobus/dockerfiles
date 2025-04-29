FROM rockylinux/rockylinux:9.5-minimal AS build
RUN microdnf install -y tar
RUN curl -O -L https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel93-8.0.7.tgz
RUN tar zxf mongodb-linux-x86_64-rhel93-8.0.7.tgz 
RUN mv mongodb-linux-x86_64-rhel93-8.0.7 mongodb
COPY rocky/9/mongo/mongod.conf mongodb/

FROM rockylinux/rockylinux:9.5-minimal
COPY --from=build /mongodb /usr/local/mongodb
CMD ["/usr/local/mongodb/bin/mongod", "--config", "/usr/local/mongodb/mongod.conf"]
EXPOSE 27017

