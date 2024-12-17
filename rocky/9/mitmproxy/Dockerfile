FROM rockylinux/rockylinux:9.5-minimal AS build
ENV VERSION 11.0.2
RUN microdnf install -y curl tar
RUN curl -O -L https://downloads.mitmproxy.org/${VERSION}/mitmproxy-${VERSION}-linux-x86_64.tar.gz 
RUN cd /mnt && tar zxf /mitmproxy-${VERSION}-linux-x86_64.tar.gz

FROM rockylinux/rockylinux:9.5-minimal
COPY --from=build /mnt /usr/local/bin
EXPOSE 1080 8080
CMD ["mitmproxy"]
