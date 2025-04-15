FROM rockylinux/rockylinux:9.5-minimal AS build
ENV NGINX_VERSION 1.24.0
ENV NJS_VERSION 0.7.12
RUN microdnf install -y autoconf automake gcc gcc-c++ make tar which zlib-devel openssl-devel pcre-devel perl-devel libxml2-devel libxslt-devel
RUN curl -O -L https://github.com/nginx/njs/archive/refs/tags/${NJS_VERSION}.tar.gz \
  && tar zxf ${NJS_VERSION}.tar.gz
RUN curl -O https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz \
  && tar -zxf nginx-${NGINX_VERSION}.tar.gz \
  && cd nginx-${NGINX_VERSION} \
  && sed -i 's:CFLAGS="$CFLAGS -g":# CFLAGS="$CFLAGS -g":g' auto/cc/gcc \
  && ./configure --prefix=/usr/local/nginx \
  --with-http_stub_status_module --with-http_ssl_module --with-http_gzip_static_module --with-http_sub_module \
  --with-stream --with-stream_ssl_module --with-http_v2_module --with-http_v3_module \
  --with-http_mp4_module --with-http_secure_link_module --with-http_auth_request_module \
  --with-threads --with-file-aio \
  --add-module=../njs-${NJS_VERSION}/nginx \
  && make -j4 \
  && make install

FROM rockylinux/rockylinux:9.5-minimal
RUN microdnf install -y zlib openssl pcre && microdnf clean all
COPY --from=build /usr/local/nginx /usr/local/nginx
ENV PATH /usr/local/nginx/sbin/:$PATH
EXPOSE 80 443
CMD ["nginx", "-g", "daemon off;"]
