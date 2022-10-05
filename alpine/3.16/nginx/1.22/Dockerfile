FROM alpine:3.16.2 AS build
WORKDIR /
ENV NGINX_VERSION 1.22.0
RUN apk add --no-cache alpine-sdk build-base pcre-dev openssl-dev zlib-dev

RUN wget https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz \
  && tar -zxf nginx-${NGINX_VERSION}.tar.gz \
  && cd nginx-${NGINX_VERSION} \
  && sed -i 's:CFLAGS="$CFLAGS -g":# CFLAGS="$CFLAGS -g":g' auto/cc/gcc \
  && ./configure --prefix=/usr/local/nginx \
  --with-http_stub_status_module --with-http_ssl_module --with-http_gzip_static_module --with-http_sub_module \
  --with-stream --with-stream_ssl_module --with-http_v2_module \
  --with-http_mp4_module --with-http_secure_link_module --with-http_auth_request_module \
  --with-threads \
  && make -j4 \
  && make install

FROM alpine:3.16.2
WORKDIR /
RUN apk add --no-cache pcre openssl zlib
COPY --from=build /usr/local/nginx /usr/local/nginx
ENV PATH /usr/local/nginx/sbin/:$PATH
EXPOSE 80 443
CMD ["nginx", "-g", "daemon off;"]
