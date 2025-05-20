FROM rockylinux/rockylinux:9.5-minimal AS build
RUN microdnf install -y zip unzip wget java-17-openjdk
RUN wget https://github.com/NotThorny/Grasscutter/releases/download/culti-aio/GrasscutterLunaGC54.zip
RUN unzip GrasscutterLunaGC54.zip -d /mnt
RUN java -jar /mnt/grasscutter.jar || true
RUN sed -i 's/localhost:27017/mongo:27017/g' /mnt/config.json

FROM rockylinux/rockylinux:9.5-minimal
WORKDIR /mnt
RUN microdnf install -y java-17-openjdk
COPY --from=build /mnt /mnt
CMD ["/usr/bin/java", "-jar", "/mnt/grasscutter.jar"]
EXPOSE 80 443 8888 22102
