ARG BASE_ROCKY_VERSION=9-minimal

FROM ghcr.io/tikobus/rockylinux:${BASE_ROCKY_VERSION} AS build
RUN microdnf install -y zip unzip
RUN curl -O -L https://github.com/NotThorny/Grasscutter/releases/download/culti-aio/GrasscutterLunaGC50.zip
RUN unzip GrasscutterLunaGC50.zip -d /mnt

FROM ghcr.io/tikobus/rockylinux:${BASE_ROCKY_VERSION}
RUN microdnf install -y java-17-openjdk
COPY --from=build /mnt /mnt
CMD ["/usr/bin/java", "-jar", "/mnt/grasscutter.jar"]
EXPOSE 443
