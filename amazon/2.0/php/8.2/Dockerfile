FROM amazonlinux:2.0.20230207.0 AS build
ENV PHP_VERSION 8.2.3
RUN yum install -y autoconf automake gcc gcc-c++ make tar gzip gunzip
RUN yum install -y curl-devel libicu-devel libjpeg-devel libpng-devel librdkafka-devel libwebp-devel libxml2-devel libzip2-devel libsodium-devel freetype-devel
RUN yum install -y libevent-devel oniguruma-devel openssl-devel readline-devel sqlite-devel zlib-devel 
RUN curl -O https://www.php.net/distributions/php-${PHP_VERSION}.tar.gz 
RUN tar zxf php-${PHP_VERSION}.tar.gz
RUN export PKG_CONFIG_PATH=PKG_CONFIG_PATH:"/usr/local/lib/pkgconfig/" \
    && cd php-${PHP_VERSION} \
    && ./configure --prefix=/usr/local/php \
    --with-curl --with-freetype --with-gettext --with-iconv-dir --with-jpeg --with-openssl --with-pear --with-readline --with-sodium --with-xmlrpc --with-zip --with-zlib \
    --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd \
    --enable-bcmath --enable-calendar --enable-exif --enable-embed --enable-fpm --enable-ftp --enable-gd --enable-intl --enable-mbstring --enable-mysqlnd --enable-pcntl \
    --enable-soap --enable-sockets
RUN cd php-${PHP_VERSION} && make -j4
RUN cd php-${PHP_VERSION} && make install

RUN export VERSION=5.3.7 \
    && curl -O https://pecl.php.net/get/redis-${VERSION}.tgz \
    && tar zxf redis-${VERSION}.tgz \
    && cd redis-${VERSION} \
    && /usr/local/php/bin/phpize \
    && ./configure --with-php-config=/usr/local/php/bin/php-config && make -j4 && make install

RUN export VERSION=3.0.8 \
    && curl -O https://pecl.php.net/get/event-${VERSION}.tgz \
    && tar zxf event-${VERSION}.tgz \
    && cd event-${VERSION} \
    && /usr/local/php/bin/phpize \
    && ./configure --with-php-config=/usr/local/php/bin/php-config && make -j4 && make install

RUN export VERSION=5.1.21 \
    && curl -O https://pecl.php.net/get/apcu-${VERSION}.tgz \
    && tar zxf apcu-${VERSION}.tgz \
    && cd apcu-${VERSION} \
    && /usr/local/php/bin/phpize \
    && ./configure --with-php-config=/usr/local/php/bin/php-config && make -j4 && make install

RUN export VERSION=3.18.3 \
    && curl -O https://pecl.php.net/get/protobuf-${VERSION}.tgz \
    && tar zxf protobuf-${VERSION}.tgz \
    && cd protobuf-${VERSION} \
    && /usr/local/php/bin/phpize \
    && ./configure --with-php-config=/usr/local/php/bin/php-config && make -j4 && make install

RUN export VERSION=6.0.3 \
    && curl -O https://pecl.php.net/get/rdkafka-${VERSION}.tgz \
    && tar zxf rdkafka-${VERSION}.tgz \
    && cd rdkafka-${VERSION} \
    && /usr/local/php/bin/phpize \
    && ./configure --with-php-config=/usr/local/php/bin/php-config && make -j4 && make install

RUN export VERSION=1.48.1 \
    && curl -O https://pecl.php.net/get/grpc-${VERSION}.tgz \
    && tar zxf grpc-${VERSION}.tgz \
    && cd grpc-${VERSION} \
    && /usr/local/php/bin/phpize \
    && ./configure --with-php-config=/usr/local/php/bin/php-config && make -j4 && make install

RUN cd /usr/local/php && strip bin/php && strip bin/php-cgi && strip bin/phpdbg && strip sbin/php-fpm && strip lib/libphp.so
RUN strip /usr/local/php/lib/php/extensions/no-debug-non-zts-*/*.so
RUN cd /usr/local/php/etc && cp php-fpm.conf.default php-fpm.conf && cp php-fpm.d/www.conf.default php-fpm.d/www.conf
RUN cp php-${PHP_VERSION}/php.ini-production /usr/local/php/lib/php.ini
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

# Install Composer
RUN /usr/local/php/bin/php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
  && /usr/local/php/bin/php -r "if (hash_file('SHA384', 'composer-setup.php') === rtrim(file_get_contents('https://composer.github.io/installer.sig'))) { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;" \
  && /usr/local/php/bin/php composer-setup.php \
  && /usr/local/php/bin/php -r "unlink('composer-setup.php');" \
  && mv composer.phar /composer

FROM amazonlinux:2.0.20230207.0
RUN yum install -y curl freetype libevent libxml2 libpng libjpeg libicu librdkafka libsodium libzip oniguruma openssl readline zlib && yum clean all
COPY --from=build /usr/local/php /usr/local/php
COPY --from=build /composer /usr/local/bin/composer
ENV PATH /usr/local/php/bin/:/usr/local/php/sbin/:$PATH
CMD [ "php-fpm", "-F" ]
