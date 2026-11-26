ARG BASE_ROCKY_VERSION=9-minimal

FROM ghcr.io/tikobus/rockylinux:${BASE_ROCKY_VERSION} AS build
ENV PHP_VERSION 8.2.36
RUN microdnf --enablerepo=devel install -y autoconf automake curl-devel freetype-devel gcc gcc-c++ libicu-devel libjpeg-devel libpng-devel librdkafka-devel libtool libwebp-devel libxml2-devel libzip-devel make oniguruma-devel openssl-devel pkgconf-pkg-config readline-devel sqlite-devel tar unzip zlib-devel 
RUN microdnf install -y epel-release
RUN microdnf --enablerepo=epel install -y libsodium-devel

RUN curl -O https://www.php.net/distributions/php-${PHP_VERSION}.tar.gz 
RUN tar zxf php-${PHP_VERSION}.tar.gz
RUN microdnf --enablerepo=devel install -y libevent-devel
RUN export PKG_CONFIG_PATH=PKG_CONFIG_PATH:"/usr/local/lib/pkgconfig/" \
    && cd php-${PHP_VERSION} \
    && ./configure --prefix=/usr/local/php \
    --with-curl --with-freetype --with-gettext --with-iconv-dir --with-jpeg --with-openssl --with-pear --with-readline --with-sodium --with-xmlrpc --with-zip --with-zlib \
    --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd \
    --enable-bcmath --enable-calendar --enable-exif --enable-embed --enable-fpm --enable-ftp --enable-gd --enable-intl --enable-mbstring --enable-mysqlnd --enable-pcntl \
    --enable-soap --enable-sockets
RUN cd php-${PHP_VERSION} && make -j4
RUN cd php-${PHP_VERSION} && make install

# Install extensions with PIE (the official PHP extension installer, replaces PECL)
# ref: https://github.com/php/pie
RUN curl -sL https://github.com/php/pie/releases/download/1.5.5/pie.phar -o /pie.phar && chmod +x /pie.phar
# PIE does not pass -j to make, so parallelize via MAKEFLAGS
ENV MAKEFLAGS=-j4
# php.ini is configured manually below, so skip PIE's auto-enable to keep it deterministic.
# Each extension is best-effort: a failed build only emits a CI warning and the build
# continues, so one extension not yet compatible with this PHP version cannot break the image.
RUN /usr/local/php/bin/php /pie.phar install --with-php-config=/usr/local/php/bin/php-config --skip-enable-extension phpredis/phpredis:6.1.0 \
    || echo "::warning::phpredis/phpredis:6.1.0 build failed, skipped"
RUN /usr/local/php/bin/php /pie.phar install --with-php-config=/usr/local/php/bin/php-config --skip-enable-extension osmanov/pecl-event:3.1.5 \
    || echo "::warning::osmanov/pecl-event:3.1.5 build failed, skipped"
RUN /usr/local/php/bin/php /pie.phar install --with-php-config=/usr/local/php/bin/php-config --skip-enable-extension apcu/apcu:5.1.24 \
    || echo "::warning::apcu/apcu:5.1.24 build failed, skipped"
RUN /usr/local/php/bin/php /pie.phar install --with-php-config=/usr/local/php/bin/php-config --skip-enable-extension rdkafka/rdkafka:6.0.5 \
    || echo "::warning::rdkafka/rdkafka:6.0.5 build failed, skipped"
RUN /usr/local/php/bin/php /pie.phar install --with-php-config=/usr/local/php/bin/php-config --skip-enable-extension grpc/grpc-php-ext:1.78.2 \
    || echo "::warning::grpc/grpc-php-ext:1.78.2 build failed, skipped"

# protobuf has no official PIE package yet (only third-party mirrors like extport/protobuf),
# so keep building it from the official PECL source for now
RUN export VERSION=3.18.3 \
    && curl -O https://pecl.php.net/get/protobuf-${VERSION}.tgz \
    && tar zxf protobuf-${VERSION}.tgz \
    && cd protobuf-${VERSION} \
    && /usr/local/php/bin/phpize \
    && ./configure --with-php-config=/usr/local/php/bin/php-config && make -j4 && make install \
    || echo "::warning::protobuf build failed, skipped"

RUN cd /usr/local/php && strip bin/php && strip bin/php-cgi && strip bin/phpdbg && strip sbin/php-fpm && strip lib/libphp.so
RUN strip /usr/local/php/lib/php/extensions/no-debug-non-zts-*/*.so
RUN cd /usr/local/php/etc && cp php-fpm.conf.default php-fpm.conf && cp php-fpm.d/www.conf.default php-fpm.d/www.conf
RUN cp php-${PHP_VERSION}/php.ini-production /usr/local/php/lib/php.ini
RUN sed -i 's/;date.timezone =/date.timezone = PRC/g' /usr/local/php/lib/php.ini
RUN sed -i 's/expose_php = On/expose_php = Off/g' /usr/local/php/lib/php.ini
RUN sed -i 's/;pid = run/pid = run/g' /usr/local/php/etc/php-fpm.conf
RUN sed -i "s/listen = 127.0.0.1:9000/listen = 0.0.0.0:9000/" /usr/local/php/etc/php-fpm.d/www.conf
# All extensions are installed but disabled by default: only commented-out
# entries are appended to php.ini. Uncomment the "extension = ..." line to
# enable an extension. An entry is only added when the .so actually exists,
# so php.ini stays clean when an extension build is skipped.
RUN ls /usr/local/php/lib/php/extensions/*/opcache.so >/dev/null 2>&1 \
    && printf '\n[Opcache]\n;zend_extension = opcache.so\n;opcache.enable = 1\n;opcache.enable_cli = 1\n' >> /usr/local/php/lib/php.ini \
    || echo "::warning::opcache.so not built, ini entry skipped"
RUN ls /usr/local/php/lib/php/extensions/*/redis.so >/dev/null 2>&1 \
    && printf '\n[Redis]\n;extension = redis.so\n' >> /usr/local/php/lib/php.ini \
    || echo "::warning::redis.so not built, ini entry skipped"
RUN ls /usr/local/php/lib/php/extensions/*/event.so >/dev/null 2>&1 \
    && printf '\n[Event]\n;extension = event.so\n' >> /usr/local/php/lib/php.ini \
    || echo "::warning::event.so not built, ini entry skipped"
RUN ls /usr/local/php/lib/php/extensions/*/apcu.so >/dev/null 2>&1 \
    && printf '\n[Apcu]\n;extension = apcu.so\n' >> /usr/local/php/lib/php.ini \
    || echo "::warning::apcu.so not built, ini entry skipped"
RUN ls /usr/local/php/lib/php/extensions/*/protobuf.so >/dev/null 2>&1 \
    && printf '\n[Protobuf]\n;extension = protobuf.so\n' >> /usr/local/php/lib/php.ini \
    || echo "::warning::protobuf.so not built, ini entry skipped"
RUN ls /usr/local/php/lib/php/extensions/*/rdkafka.so >/dev/null 2>&1 \
    && printf '\n[Rdkafka]\n;extension = rdkafka.so\n' >> /usr/local/php/lib/php.ini \
    || echo "::warning::rdkafka.so not built, ini entry skipped"
RUN ls /usr/local/php/lib/php/extensions/*/grpc.so >/dev/null 2>&1 \
    && printf '\n[Grpc]\n;extension = grpc.so\n' >> /usr/local/php/lib/php.ini \
    || echo "::warning::grpc.so not built, ini entry skipped"

# Install Composer
RUN /usr/local/php/bin/php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
  && /usr/local/php/bin/php -r "if (hash_file('SHA384', 'composer-setup.php') === rtrim(file_get_contents('https://composer.github.io/installer.sig'))) { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;" \
  && /usr/local/php/bin/php composer-setup.php \
  && /usr/local/php/bin/php -r "unlink('composer-setup.php');" \
  && mv composer.phar /composer

FROM ghcr.io/tikobus/rockylinux:${BASE_ROCKY_VERSION}
RUN microdnf install -y epel-release && microdnf clean all
RUN microdnf --enablerepo=epel install -y curl freetype libevent libxml2 libpng libjpeg libicu librdkafka libsodium libzip oniguruma openssl readline zlib && microdnf clean all
COPY --from=build /usr/local/php /usr/local/php
COPY --from=build /composer /usr/local/bin/composer
COPY --from=build /pie.phar /usr/local/bin/pie
ENV PATH /usr/local/php/bin/:/usr/local/php/sbin/:$PATH
CMD [ "php-fpm", "-F" ]
