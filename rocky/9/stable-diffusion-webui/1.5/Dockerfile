FROM rockylinux/rockylinux:9.5-minimal AS build
ENV VERSION 1.10.1
RUN microdnf install -y wget git python3 python3-pip curl
RUN mkdir /app
RUN cd /app && git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git
RUN cd /app/stable-diffusion-webui && git checkout v${VERSION}
RUN cd /app/stable-diffusion-webui && python3 -m venv venv && source venv/bin/activate && pip3 install -r requirements.txt && pip3 install xformers
RUN cd /app/stable-diffusion-webui && sed -i 's/can_run_as_root=0/can_run_as_root=1/' webui.sh
RUN cd /app/stable-diffusion-webui && curl -o models/Stable-diffusion/v1-5-pruned-emaonly.safetensors https://huggingface.co/runwayml/stable-diffusion-v1-5/resolve/main/v1-5-pruned-emaonly.safetensors

FROM rockylinux/rockylinux:9.5-minimal
WORKDIR /app/stable-diffusion-webui
RUN microdnf install -y python3 git
COPY --from=build /app /app
EXPOSE 7860
CMD ["./webui.sh", "--skip-torch-cuda-test", "--skip-version-check"]
