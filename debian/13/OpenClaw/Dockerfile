FROM docker.io/library/node:24
ENV CLAW_VERSION 2026.4.11
RUN npm i -g openclaw@${CLAW_VERSION} && npm cache clean --force
EXPOSE 18789
