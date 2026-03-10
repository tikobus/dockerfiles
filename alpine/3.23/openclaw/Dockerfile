FROM alpine:3.23.3
WORKDIR /
ENV CLAW_VERSION 2026.3.8
RUN apk add nodejs@20 npm@20 git
RUN npm i -g openclaw && npm cache clean
CMD ["openclaw", "onboard", "--install-daemon"]
EXPOSE 18789
