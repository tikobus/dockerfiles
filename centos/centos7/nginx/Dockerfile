FROM centos:centos7.9.2009 AS build
WORKDIR /usr/local
RUN yum install -y gcc make zlib-devel openssl-devel pcre-devel perl-devel

RUN export VERSION=1.1.1o \
  && curl -O https://www.openssl.org/source/openssl-${VERSION}.tar.gz \
  && tar -zxf openssl-${VERSION}.tar.gz

RUN export VERSION=1.22.0 \
  && curl -O https://nginx.org/download/nginx-${VERSION}.tar.gz \
  && tar -zxf nginx-${VERSION}.tar.gz \
  && cd nginx-${VERSION} \
  && sed -i 's:CFLAGS="$CFLAGS -g":# CFLAGS="$CFLAGS -g":g' auto/cc/gcc \
  && ./configure --prefix=/usr/local/nginx \
  --with-openssl=../openssl-1.1.1o \
  --with-http_stub_status_module --with-http_ssl_module --with-http_gzip_static_module --with-http_sub_module \
  --with-stream --with-stream_ssl_module --with-http_v2_module \
  --with-http_mp4_module --with-http_secure_link_module --with-http_auth_request_module \
  --with-threads --with-file-aio \
  && make -j4 \
  && make install

FROM centos:centos7.9.2009
WORKDIR /
RUN yum install -y zlib openssl pcre && yum clean all
COPY --from=build /usr/local/nginx /usr/local/nginx
ENV PATH /usr/local/nginx/sbin/:$PATH
CMD ["nginx", "-g", "daemon off;"]
