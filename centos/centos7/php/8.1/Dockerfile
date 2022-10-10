FROM centos:centos7.9.2009 AS build
WORKDIR /usr/local
RUN yum install -y gcc gcc-c++ make perl autoconf automake libtool openssl-devel curl-devel libevent-devel libxml2-devel libjpeg-devel librdkafka-devel libpng-devel readline-devel freetype-devel zlib-devel libicu-devel sqlite-devel centos-release-scl

RUN export VERSION=1.3.2 \
  && curl -O https://libzip.org/download/libzip-${VERSION}.tar.gz \
  && tar -zxvf libzip-${VERSION}.tar.gz \
  && cd libzip-${VERSION} \
  && ./configure \
  && make \
  && make install

RUN export VERSION=6.9.4 \
  && curl -o oniguruma-${VERSION}.tar.gz -L https://github.com/kkos/oniguruma/archive/v${VERSION}.tar.gz \
  && tar -xvf oniguruma-${VERSION}.tar.gz \
  && cd oniguruma-${VERSION} \
  && ./autogen.sh \
  && ./configure \
  && make \
  && make install

RUN export VERSION=8.1.10 \
  && curl -C - -O https://www.php.net/distributions/php-${VERSION}.tar.gz \
  && tar -zxf php-${VERSION}.tar.gz

RUN export PKG_CONFIG_PATH=PKG_CONFIG_PATH:"/usr/local/lib/pkgconfig/" \
  && cd php-* \
  && ./configure --prefix=/usr/local/php \
  --with-curl --with-freetype --with-gettext --with-iconv-dir --with-jpeg --with-openssl --with-pear --with-readline --with-xmlrpc --with-zip --with-zlib \
  --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd \
  --enable-bcmath --enable-calendar --enable-exif --enable-embed --enable-fpm --enable-ftp --enable-gd --enable-intl --enable-mbstring --enable-mysqlnd --enable-pcntl \
  --enable-soap --enable-sockets

RUN cd php-* && make

RUN cd php-* && make install

RUN export VERSION=5.3.7 \
  && curl -O https://pecl.php.net/get/redis-${VERSION}.tgz \
  && tar zxf redis-${VERSION}.tgz \
  && cd redis-${VERSION} \
  && /usr/local/php/bin/phpize \
  && ./configure --with-php-config=/usr/local/php/bin/php-config && make && make install

RUN export VERSION=3.0.8 \
  && curl -O https://pecl.php.net/get/event-${VERSION}.tgz \
  && tar zxf event-${VERSION}.tgz \
  && cd event-${VERSION} \
  && /usr/local/php/bin/phpize \
  && ./configure --with-php-config=/usr/local/php/bin/php-config && make && make install

RUN export VERSION=5.1.21 \
  && curl -O https://pecl.php.net/get/apcu-${VERSION}.tgz \
  && tar zxf apcu-${VERSION}.tgz \
  && cd apcu-${VERSION} \
  && /usr/local/php/bin/phpize \
  && ./configure --with-php-config=/usr/local/php/bin/php-config && make && make install

RUN export VERSION=3.18.3 \
  && curl -O https://pecl.php.net/get/protobuf-${VERSION}.tgz \
  && tar zxf protobuf-${VERSION}.tgz \
  && cd protobuf-${VERSION} \
  && /usr/local/php/bin/phpize \
  && ./configure --with-php-config=/usr/local/php/bin/php-config && make && make install

RUN yum install -y librdkafka-devel

RUN export VERSION=6.0.3 \
  && curl -O https://pecl.php.net/get/rdkafka-${VERSION}.tgz \
  && tar zxf rdkafka-${VERSION}.tgz \
  && cd rdkafka-${VERSION} \
  && /usr/local/php/bin/phpize \
  && ./configure --with-php-config=/usr/local/php/bin/php-config && make && make install

RUN yum install -y devtoolset-8-gcc*

RUN export VERSION=1.48.1 \
  && scl enable devtoolset-8 bash && source scl_source enable devtoolset-8 \
  && curl -O https://pecl.php.net/get/grpc-${VERSION}.tgz \
  && tar zxf grpc-${VERSION}.tgz \
  && cd grpc-${VERSION} \
  && /usr/local/php/bin/phpize \
  && ./configure --with-php-config=/usr/local/php/bin/php-config && make && make install

RUN cd /usr/local/php && strip bin/php && strip bin/php-cgi && strip bin/phpdbg && strip sbin/php-fpm && strip lib/libphp.so
RUN cd /usr/local/php/etc && cp php-fpm.conf.default php-fpm.conf && cp php-fpm.d/www.conf.default php-fpm.d/www.conf
RUN cp /usr/local/php-*/php.ini-production /usr/local/php/lib/php.ini
RUN sed -i 's/;date.timezone =/date.timezone = PRC/g' /usr/local/php/lib/php.ini
RUN sed -i 's/expose_php = On/expose_php = Off/g' /usr/local/php/lib/php.ini
RUN sed -i 's/;pid = run/pid = run/g' /usr/local/php/etc/php-fpm.conf
RUN sed -i "s/listen = 127.0.0.1:9000/listen = 0.0.0.0:9000/" /usr/local/php/etc/php-fpm.d/www.conf
RUN echo -e "\n[Opcache] \nzend_extension = opcache.so \nopcache.enable = 1 \nopcache.enable_cli = 1 \n" >> /usr/local/php/lib/php.ini
RUN echo -e "\n[Redis]\nextension = redis.so\n" >> /usr/local/php/lib/php.ini
RUN echo -e "\n[Event]\nextension = event.so\n" >> /usr/local/php/lib/php.ini
RUN echo -e "\n[Apcu]\nextension = apcu.so\n" >> /usr/local/php/lib/php.ini
RUN echo -e "\n[Protobuf]\nextension = protobuf.so\n" >> /usr/local/php/lib/php.ini
RUN echo -e "\n[Rdkafka]\nextension = rdkafka.so\n" >> /usr/local/php/lib/php.ini
RUN echo -e "\n[Grpc]\nextension = grpc.so\n" >> /usr/local/php/lib/php.ini
RUN mkdir /usr/local-build
RUN cp -r /usr/local/php /usr/local-build/
RUN cp -r /usr/local/lib /usr/local-build/

FROM centos:centos7.9.2009
WORKDIR /usr/local/
RUN yum install -y autoconf automake openssl curl libevent libxml2 libpng libjpeg readline freetype zlib libicu librdkafka && yum clean all
COPY --from=build /usr/local-build /usr/local
ENV PATH /usr/local/php/bin/:/usr/local/php/sbin/:$PATH
