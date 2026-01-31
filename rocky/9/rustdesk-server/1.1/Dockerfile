FROM rockylinux/rockylinux:9.6-minimal AS build
RUN microdnf install -y wget unzip
ENV RustDesk_VERSION 1.1.15
RUN wget https://github.com/rustdesk/rustdesk-server/releases/download/${RustDesk_VERSION}/rustdesk-server-linux-amd64.zip
RUN mkdir /usr/local/rustdesk && unzip rustdesk-*.zip && mv amd64/* /usr/local/rustdesk/

FROM rockylinux/rockylinux:9.6-minimal
WORKDIR /usr/local/rustdesk
COPY --from=build /usr/local/rustdesk /usr/local/rustdesk
ENV PATH /usr/local/rustdesk/:$PATH
EXPOSE 21115 21116 21118
CMD ["/usr/local/rustdesk/hbbs"]
