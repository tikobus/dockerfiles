FROM alpine:3
WORKDIR /
ENV CLAW_VERSION 2026.5.12
RUN apk add --no-cache nodejs npm git
RUN npm i -g openclaw && npm cache clean && openclaw onboard
CMD ["openclaw", "gateway"]
EXPOSE 18789
