FROM centos:centos7.9.2009 AS build
ENV NGINX_VERSION 1.22.1
ENV NJS_VERSION 0.7.8
ENV OPENSSL_VERSION 1.1.1o
RUN yum install -y autoconf automake gcc gcc-c++ make which zlib-devel openssl-devel pcre-devel perl-devel

RUN curl -O https://www.openssl.org/source/openssl-${OPENSSL_VERSION}.tar.gz \
  && tar zxf openssl-${OPENSSL_VERSION}.tar.gz

RUN curl -O -L https://github.com/nginx/njs/archive/refs/tags/${NJS_VERSION}.tar.gz \
  && tar zxf ${NJS_VERSION}.tar.gz

RUN curl -O https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz \
  && tar zxf nginx-${NGINX_VERSION}.tar.gz \
  && cd nginx-${NGINX_VERSION} \
  && sed -i 's:CFLAGS="$CFLAGS -g":# CFLAGS="$CFLAGS -g":g' auto/cc/gcc \
  && ./configure --prefix=/usr/local/nginx \
  --with-openssl=../openssl-1.1.1o \
  --with-http_stub_status_module --with-http_ssl_module --with-http_gzip_static_module --with-http_sub_module \
  --with-stream --with-stream_ssl_module --with-http_v2_module \
  --with-http_mp4_module --with-http_secure_link_module --with-http_auth_request_module \
  --with-threads --with-file-aio \
  --add-module=../njs-${NJS_VERSION}/nginx \
  && make -j4 \
  && make install

FROM centos:centos7.9.2009
RUN yum install -y zlib openssl pcre && yum clean all
COPY --from=build /usr/local/nginx /usr/local/nginx
ENV PATH /usr/local/nginx/sbin/:$PATH
EXPOSE 80 443
CMD ["nginx", "-g", "daemon off;"]
