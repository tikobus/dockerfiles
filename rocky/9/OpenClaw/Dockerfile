FROM rockylinux/rockylinux:9.7-minimal
ENV PATH=/usr/local/node/bin/:$PATH
RUN microdnf install -y wget xz tar git make cmake g++ && wget https://nodejs.org/dist/v22.22.1/node-v22.22.1-linux-x64.tar.xz && tar -xJf node-v22.22.1-linux-x64.tar.xz && mv node-v22.22.1-linux-x64 /usr/local/node && /usr/local/node/bin/npm i -g openclaw@2026.3.8
CMD ["openclaw", "onboard", "--install-daemon"]
EXPOSE 18789
