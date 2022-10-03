FROM alpine:3.16.2 AS build
ENV PHP_VERSION 8.1.11
RUN apk add gcc g++ make perl autoconf automake 
RUN apk add openssl-dev curl-dev libevent-dev libxml2-dev librdkafka-dev libpng-dev 
RUN apk add readline-dev freetype-dev zlib-dev sqlite-dev libzip-dev oniguruma-dev libjpeg-turbo-dev
RUN apk add libintl musl-libintl icu-dev

RUN wget https://www.php.net/distributions/php-${PHP_VERSION}.tar.gz \
  && tar -zxf php-${PHP_VERSION}.tar.gz

RUN export PKG_CONFIG_PATH=PKG_CONFIG_PATH:"/usr/local/lib/pkgconfig/" \
  && cd php-* \
  && ./configure --prefix=/usr/local/php \
  --with-curl --with-freetype --with-gettext --with-jpeg --with-openssl --with-pear --with-readline --with-zip --with-zlib \
  --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd \
  --enable-bcmath --enable-calendar --enable-exif --enable-embed --enable-fpm --enable-ftp --enable-gd --enable-intl --enable-mbstring --enable-mysqlnd --enable-pcntl \
  --enable-soap --enable-sockets

RUN cd php-* && make

RUN cd php-* && make install

RUN export VERSION=5.3.7 \
  && wget https://pecl.php.net/get/redis-${VERSION}.tgz \
  && tar zxf redis-${VERSION}.tgz \
  && cd redis-${VERSION} \
  && /usr/local/php/bin/phpize \
  && ./configure --with-php-config=/usr/local/php/bin/php-config && make && make install

RUN export VERSION=3.0.8 \
  && wget https://pecl.php.net/get/event-${VERSION}.tgz \
  && tar zxf event-${VERSION}.tgz \
  && cd event-${VERSION} \
  && /usr/local/php/bin/phpize \
  && ./configure --with-php-config=/usr/local/php/bin/php-config && make && make install

RUN export VERSION=5.1.21 \
  && wget https://pecl.php.net/get/apcu-${VERSION}.tgz \
  && tar zxf apcu-${VERSION}.tgz \
  && cd apcu-${VERSION} \
  && /usr/local/php/bin/phpize \
  && ./configure --with-php-config=/usr/local/php/bin/php-config && make && make install

RUN export VERSION=3.18.3 \
  && wget https://pecl.php.net/get/protobuf-${VERSION}.tgz \
  && tar zxf protobuf-${VERSION}.tgz \
  && cd protobuf-${VERSION} \
  && /usr/local/php/bin/phpize \
  && ./configure --with-php-config=/usr/local/php/bin/php-config && make && make install

RUN export VERSION=6.0.3 \
  && wget https://pecl.php.net/get/rdkafka-${VERSION}.tgz \
  && tar zxf rdkafka-${VERSION}.tgz \
  && cd rdkafka-${VERSION} \
  && /usr/local/php/bin/phpize \
  && ./configure --with-php-config=/usr/local/php/bin/php-config && make && make install

RUN cd /usr/local/php && strip bin/php && strip bin/php-cgi && strip bin/phpdbg && strip sbin/php-fpm && strip lib/libphp.so
RUN cd /usr/local/php/etc && cp php-fpm.conf.default php-fpm.conf && cp php-fpm.d/www.conf.default php-fpm.d/www.conf
RUN cp php-*/php.ini-production /usr/local/php/lib/php.ini
RUN sed -i 's/;date.timezone =/date.timezone = PRC/g' /usr/local/php/lib/php.ini
RUN sed -i 's/expose_php = On/expose_php = Off/g' /usr/local/php/lib/php.ini
RUN sed -i 's/;pid = run/pid = run/g' /usr/local/php/etc/php-fpm.conf
RUN echo -e "\n[Opcache] \nzend_extension = opcache.so \nopcache.enable = 1 \nopcache.enable_cli = 1 \n" >> /usr/local/php/lib/php.ini
RUN echo -e "\n[Redis]\nextension = redis.so\n" >> /usr/local/php/lib/php.ini
RUN echo -e "\n[Event]\nextension = event.so\n" >> /usr/local/php/lib/php.ini
RUN echo -e "\n[Apcu]\nextension = apcu.so\n" >> /usr/local/php/lib/php.ini
RUN echo -e "\n[Protobuf]\nextension = protobuf.so\n" >> /usr/local/php/lib/php.ini
RUN echo -e "\n[Rdkafka]\nextension = rdkafka.so\n" >> /usr/local/php/lib/php.ini
RUN mkdir /usr/local-build
RUN cp -r /usr/local/php /usr/local-build/
RUN cp -r /usr/local/lib /usr/local-build/

FROM alpine:3.16.2
RUN apk add autoconf automake openssl curl libevent libxml2 libpng libjpeg readline freetype zlib librdkafka libzip oniguruma
COPY --from=build /usr/local-build /usr/local
ENV PATH /usr/local/php/bin/:/usr/local/php/sbin/:$PATH
