FROM rockylinux:9.3.20231119-minimal AS build
ENV PHP_VERSION 8.4.0
ENV INSTALL_PATH ${INSTALL_PATH}/8.4
RUN microdnf install -y epel-release
RUN microdnf --enablerepo=devel install -y autoconf automake curl-devel freetype-devel gcc gcc-c++ libicu-devel libjpeg-devel libpng-devel librdkafka-devel libwebp-devel libxml2-devel libzip-devel make oniguruma-devel openssl-devel readline-devel sqlite-devel tar zlib-devel libevent-devel
RUN microdnf --enablerepo=epel install -y libsodium-devel

RUN curl -O https://www.php.net/distributions/php-${PHP_VERSION}.tar.gz 
RUN tar zxf php-${PHP_VERSION}.tar.gz
RUN export PKG_CONFIG_PATH=PKG_CONFIG_PATH:"/usr/local/lib/pkgconfig/" \
    && cd php-${PHP_VERSION} \
    && ./configure --prefix=${INSTALL_PATH} \
    --with-curl --with-freetype --with-gettext --with-jpeg --with-openssl --with-pear --with-readline --with-sodium --with-zip --with-zlib \
    --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd \
    --enable-bcmath --enable-calendar --enable-exif --enable-embed --enable-fpm --enable-ftp --enable-gd --enable-intl --enable-mbstring --enable-mysqlnd --enable-pcntl \
    --enable-soap --enable-sockets
RUN cd php-${PHP_VERSION} && make -j4
RUN cd php-${PHP_VERSION} && make install

RUN export VERSION=5.3.7 \
    && curl -O https://pecl.php.net/get/redis-${VERSION}.tgz \
    && tar zxf redis-${VERSION}.tgz \
    && cd redis-${VERSION} \
    && ${INSTALL_PATH}/bin/phpize \
    && ./configure --with-php-config=${INSTALL_PATH}/bin/php-config && make -j4 && make install

RUN export VERSION=3.0.8 \
    && curl -O https://pecl.php.net/get/event-${VERSION}.tgz \
    && tar zxf event-${VERSION}.tgz \
    && cd event-${VERSION} \
    && ${INSTALL_PATH}/bin/phpize \
    && ./configure --with-php-config=${INSTALL_PATH}/bin/php-config && make -j4 && make install

RUN export VERSION=5.1.21 \
    && curl -O https://pecl.php.net/get/apcu-${VERSION}.tgz \
    && tar zxf apcu-${VERSION}.tgz \
    && cd apcu-${VERSION} \
    && ${INSTALL_PATH}/bin/phpize \
    && ./configure --with-php-config=${INSTALL_PATH}/bin/php-config && make -j4 && make install

RUN export VERSION=3.18.3 \
    && curl -O https://pecl.php.net/get/protobuf-${VERSION}.tgz \
    && tar zxf protobuf-${VERSION}.tgz \
    && cd protobuf-${VERSION} \
    && ${INSTALL_PATH}/bin/phpize \
    && ./configure --with-php-config=${INSTALL_PATH}/bin/php-config && make -j4 && make install

RUN export VERSION=6.0.3 \
    && curl -O https://pecl.php.net/get/rdkafka-${VERSION}.tgz \
    && tar zxf rdkafka-${VERSION}.tgz \
    && cd rdkafka-${VERSION} \
    && ${INSTALL_PATH}/bin/phpize \
    && ./configure --with-php-config=${INSTALL_PATH}/bin/php-config && make -j4 && make install

RUN export VERSION=1.48.1 \
    && curl -O https://pecl.php.net/get/grpc-${VERSION}.tgz \
    && tar zxf grpc-${VERSION}.tgz \
    && cd grpc-${VERSION} \
    && ${INSTALL_PATH}/bin/phpize \
    && ./configure --with-php-config=${INSTALL_PATH}/bin/php-config && make -j4 && make install

RUN cd ${INSTALL_PATH} && strip bin/php && strip bin/php-cgi && strip bin/phpdbg && strip sbin/php-fpm && strip lib/libphp.so
RUN strip ${INSTALL_PATH}/lib/php/extensions/no-debug-non-zts-*/*.so
RUN cd ${INSTALL_PATH}/etc && cp php-fpm.conf.default php-fpm.conf && cp php-fpm.d/www.conf.default php-fpm.d/www.conf
RUN cp php-${PHP_VERSION}/php.ini-production ${INSTALL_PATH}/lib/php.ini
RUN sed -i 's/;date.timezone =/date.timezone = PRC/g' ${INSTALL_PATH}/lib/php.ini
RUN sed -i 's/expose_php = On/expose_php = Off/g' ${INSTALL_PATH}/lib/php.ini
RUN sed -i 's/;pid = run/pid = run/g' ${INSTALL_PATH}/etc/php-fpm.conf
RUN sed -i "s/listen = 127.0.0.1:9000/listen = 0.0.0.0:9000/" ${INSTALL_PATH}/etc/php-fpm.d/www.conf
RUN echo -e "\n[Opcache] \nzend_extension = opcache.so \nopcache.enable = 1 \nopcache.enable_cli = 1 \n" >> ${INSTALL_PATH}/lib/php.ini
RUN echo -e "\n[Redis]\nextension = redis.so\n" >> ${INSTALL_PATH}/lib/php.ini
RUN echo -e "\n[Event]\nextension = event.so\n" >> ${INSTALL_PATH}/lib/php.ini
RUN echo -e "\n[Apcu]\nextension = apcu.so\n" >> ${INSTALL_PATH}/lib/php.ini
RUN echo -e "\n[Protobuf]\nextension = protobuf.so\n" >> ${INSTALL_PATH}/lib/php.ini
RUN echo -e "\n[Rdkafka]\nextension = rdkafka.so\n" >> ${INSTALL_PATH}/lib/php.ini
RUN echo -e "\n[Grpc]\nextension = grpc.so\n" >> ${INSTALL_PATH}/lib/php.ini

# Install Composer
RUN ${INSTALL_PATH}/bin/php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
  && ${INSTALL_PATH}/bin/php -r "if (hash_file('SHA384', 'composer-setup.php') === rtrim(file_get_contents('https://composer.github.io/installer.sig'))) { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;" \
  && ${INSTALL_PATH}/bin/php composer-setup.php \
  && ${INSTALL_PATH}/bin/php -r "unlink('composer-setup.php');" \
  && mv composer.phar /composer

FROM rockylinux:9.3.20231119-minimal
RUN microdnf install -y epel-release && microdnf clean all
RUN microdnf --enablerepo=epel install -y curl freetype libevent libxml2 libpng libjpeg libicu librdkafka libsodium libzip oniguruma openssl readline zlib && microdnf clean all
COPY --from=build ${INSTALL_PATH} ${INSTALL_PATH}
COPY --from=build /composer /usr/local/bin/composer
ENV PATH ${INSTALL_PATH}/bin/:${INSTALL_PATH}/sbin/:$PATH
CMD [ "php-fpm", "-F" ]
