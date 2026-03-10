FROM alpine:3.23.3
WORKDIR /
ENV CLAW_VERSION 2026.3.8
RUN apk add nodejs
RUN npm i -g openclaw@${CLAW_VERSION}
CMD ["openclaw", "onboard", "--install-daemon"]
EXPOSE 18789
