FROM docker.io/library/node:22
ENV CLAW_VERSION 2026.3.8
RUN npm i -g openclaw@${CLAW_VERSION}
ENV PATH /usr/local/kafka/bin/:$PATH
CMD ["openclaw", "onboard", "--install-daemon"]
EXPOSE 18789
