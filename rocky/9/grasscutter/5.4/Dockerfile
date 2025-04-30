FROM rockylinux/rockylinux:9.5-minimal AS build
RUN microdnf install -y zip unzip wget
RUN wget https://github.com/NotThorny/Grasscutter/releases/download/culti-aio/GrasscutterLunaGC54.zip
RUN unzip GrasscutterLunaGC54.zip -d /mnt

FROM rockylinux/rockylinux:9.5-minimal
RUN microdnf install -y java-17-openjdk
COPY --from=build /mnt /mnt
CMD ["/usr/bin/java", "-jar", "/mnt/grasscutter.jar"]
EXPOSE 80 443 8888 22102
