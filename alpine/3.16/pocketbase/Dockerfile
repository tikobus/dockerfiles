FROM alpine:3.16.2 AS build
ARG TARGETOS=linux
ARG TARGETARCH=amd64
ARG VERSION=0.14.0
ENV BUILDX_ARCH="${TARGETOS:-linux}_${TARGETARCH:-amd64}"
RUN apk add --no-cache ca-certificates unzip wget zip zlib-dev
RUN wget https://github.com/pocketbase/pocketbase/releases/download/v${VERSION}/pocketbase_${VERSION}_${BUILDX_ARCH}.zip && unzip pocketbase_${VERSION}_${BUILDX_ARCH}.zip && chmod +x /pocketbase

FROM alpine:3.16.2
EXPOSE 8090
COPY --from=build /pocketbase /usr/local/bin/pocketbase
CMD ["/usr/local/bin/pocketbase", "serve", "--http=0.0.0.0:8090"]
