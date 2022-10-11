FROM rockylinux:9.0.20220720-minimal AS build
RUN microdnf install -y gcc make tar zlib-devel openssl-devel pcre-devel perl-devel

RUN export VERSION=1.22.0 \
  && curl -O https://nginx.org/download/nginx-${VERSION}.tar.gz \
  && tar -zxf nginx-${VERSION}.tar.gz \
  && cd nginx-${VERSION} \
  && sed -i 's:CFLAGS="$CFLAGS -g":# CFLAGS="$CFLAGS -g":g' auto/cc/gcc \
  && ./configure --prefix=/usr/local/nginx \
  --with-http_stub_status_module --with-http_ssl_module --with-http_gzip_static_module --with-http_sub_module \
  --with-stream --with-stream_ssl_module --with-http_v2_module \
  --with-http_mp4_module --with-http_secure_link_module --with-http_auth_request_module \
  --with-threads --with-file-aio \
  && make -j4 \
  && make install

FROM rockylinux:9.0.20220720-minimal
RUN microdnf install -y zlib openssl pcre && microdnf clean all
COPY --from=build /usr/local/nginx /usr/local/nginx
ENV PATH /usr/local/nginx/sbin/:$PATH
EXPOSE 80 443
CMD ["nginx", "-g", "daemon off;"]
