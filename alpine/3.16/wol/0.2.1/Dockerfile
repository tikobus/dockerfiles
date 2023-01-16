FROM alpine:3.16.2 AS build
ENV WOL_VERSION 0.2.1
RUN apk add --no-cache go bash git binutils
RUN wget https://github.com/tikobus/wol/archive/refs/tags/v${WOL_VERSION}.zip
RUN unzip v${WOL_VERSION}.zip
RUN cd wol-${WOL_VERSION} && go build .

FROM alpine:3.16.2
COPY --from=build /wol-0.2.1/wol /usr/local/bin/
CMD ["wol"]
