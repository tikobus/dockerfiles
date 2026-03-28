FROM docker.io/library/node:24
ENV CLAW_VERSION 2026.3.24
RUN npm i -g openclaw@${CLAW_VERSION}
RUN openclaw onboard
CMD ["openclaw", "gateway"]
EXPOSE 18789
