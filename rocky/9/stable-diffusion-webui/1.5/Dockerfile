FROM rockylinux:9.2.20230513-minimal AS build
ENV VERSION 1.5.2
RUN microdnf install -y wget git python3 python3-pip
RUN mkdir /app
RUN cd /app && git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git
RUN cd /app/stable-diffusion-webui && git checkout v${VERSION}
RUN pip3 install -r requirements.txt

FROM rockylinux:9.2.20230513-minimal
RUN microdnf install -y python3
COPY --from=build /app /app
CMD ["/app/webui.sh"]
