FROM rockylinux/rockylinux:9.5-minimal AS build
RUN microdnf install -y zip unzip java-17-openjdk
RUN curl -O -L https://github.com/NotThorny/Grasscutter/releases/download/culti-aio/GrasscutterCulti.zip
RUN unzip GrasscutterCulti.zip -d /mnt
RUN java -jar /mnt/grasscutter.jar
RUN sed -i 's/localhost:27017/mongo:27017/g' /mnt/config.json

FROM rockylinux/rockylinux:9.5-minimal
RUN microdnf install -y java-17-openjdk
COPY --from=build /mnt /mnt
CMD ["/usr/bin/java", "-jar", "/mnt/grasscutter.jar"]
EXPOSE 80 443 8888 22102
