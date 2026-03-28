FROM alpine:3.23.3
WORKDIR /
ENV CLAW_VERSION 2026.3.24
RUN apk add nodejs@24 npm@24 git
RUN npm i -g openclaw && npm cache clean && openclaw onboard
CMD ["openclaw", "gateway"]
EXPOSE 18789
