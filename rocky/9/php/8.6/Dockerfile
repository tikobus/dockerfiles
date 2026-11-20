FROM ghcr.io/tikobus/rockylinux:9.8.20260525.0-minimal AS build
ENV PHP_VERSION=8.6.0
ENV INSTALL_PATH=/usr/local/php/8.6
RUN microdnf install -y epel-release
RUN microdnf --enablerepo=devel install -y autoconf automake curl-devel freetype-devel gcc gcc-c++ gmp-devel libevent-devel libicu-devel libjpeg-devel libpng-devel libpq-devel librdkafka-devel libtool libwebp-devel libxml2-devel libxslt-devel libzip-devel make oniguruma-devel openssl-devel readline-devel sqlite-devel tar unzip zlib-devel
RUN microdnf --enablerepo=epel install -y libsodium-devel libtidy-devel

RUN curl -O https://www.php.net/distributions/php-${PHP_VERSION}.tar.gz
RUN tar zxf php-${PHP_VERSION}.tar.gz
RUN export PKG_CONFIG_PATH=PKG_CONFIG_PATH:"/usr/local/lib/pkgconfig/" \
    && cd php-${PHP_VERSION} \
    && ./configure --prefix=${INSTALL_PATH} \
    --with-curl --with-freetype --with-gettext --with-gmp --with-jpeg --with-openssl --with-pear --with-readline --with-sodium --with-tidy --with-xsl --with-zip --with-zlib \
    --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --with-pgsql --with-pdo-pgsql \
    --enable-bcmath --enable-calendar --enable-exif --enable-embed --enable-fpm --enable-ftp --enable-gd --enable-intl --enable-mbstring --enable-mysqlnd --enable-pcntl \
    --enable-shmop --enable-soap --enable-sockets --enable-sysvmsg --enable-sysvsem --enable-sysvshm --enable-opcache
RUN cd php-${PHP_VERSION} && make -j4
RUN cd php-${PHP_VERSION} && make install

# Install extensions with PIE (the official PHP extension installer, replaces PECL)
# ref: https://github.com/php/pie
RUN curl -sL https://github.com/php/pie/releases/download/1.5.5/pie.phar -o /pie.phar
# PIE does not pass -j to make, so parallelize via MAKEFLAGS
ENV MAKEFLAGS=-j4
# php.ini is configured manually below, so skip PIE's auto-enable to keep it deterministic.
# Each extension is best-effort: on failure dump PIE's saved make log and keep going,
# so one extension not yet compatible with this PHP version won't kill the whole build.
RUN ${INSTALL_PATH}/bin/php /pie.phar install --with-php-config=${INSTALL_PATH}/bin/php-config --skip-enable-extension phpredis/phpredis:6.3.0 \
    || (ls -t /tmp/pie_make_output_* 2>/dev/null | head -1 | xargs -r tail -60; echo "::warning::phpredis/phpredis:6.3.0 build failed, skipped")
RUN ${INSTALL_PATH}/bin/php /pie.phar install --with-php-config=${INSTALL_PATH}/bin/php-config --skip-enable-extension osmanov/pecl-event:3.2.0 \
    || (ls -t /tmp/pie_make_output_* 2>/dev/null | head -1 | xargs -r tail -60; echo "::warning::osmanov/pecl-event:3.2.0 build failed, skipped")
RUN ${INSTALL_PATH}/bin/php /pie.phar install --with-php-config=${INSTALL_PATH}/bin/php-config --skip-enable-extension apcu/apcu:5.1.28 \
    || (ls -t /tmp/pie_make_output_* 2>/dev/null | head -1 | xargs -r tail -60; echo "::warning::apcu/apcu:5.1.28 build failed, skipped")
RUN ${INSTALL_PATH}/bin/php /pie.phar install --with-php-config=${INSTALL_PATH}/bin/php-config --skip-enable-extension rdkafka/rdkafka:6.0.5 \
    || (ls -t /tmp/pie_make_output_* 2>/dev/null | head -1 | xargs -r tail -60; echo "::warning::rdkafka/rdkafka:6.0.5 build failed, skipped")
RUN ${INSTALL_PATH}/bin/php /pie.phar install --with-php-config=${INSTALL_PATH}/bin/php-config --skip-enable-extension grpc/grpc-php-ext:1.85.3 \
    || (ls -t /tmp/pie_make_output_* 2>/dev/null | head -1 | xargs -r tail -60; echo "::warning::grpc/grpc-php-ext:1.85.3 build failed, skipped")

# protobuf has no official PIE package yet (only third-party mirrors like extport/protobuf),
# so keep building it from the official PECL source for now
RUN export VERSION=5.36.2 \
    && curl -O https://pecl.php.net/get/protobuf-${VERSION}.tgz \
    && tar zxf protobuf-${VERSION}.tgz \
    && cd protobuf-${VERSION} \
    && ${INSTALL_PATH}/bin/phpize \
    && ./configure --with-php-config=${INSTALL_PATH}/bin/php-config && make -j4 && make install \
    || echo "::warning::protobuf build failed, skipped"

RUN cd ${INSTALL_PATH} && strip bin/php && strip bin/php-cgi && strip bin/phpdbg && strip sbin/php-fpm && strip lib/libphp.so
RUN strip ${INSTALL_PATH}/lib/php/extensions/no-debug-non-zts-*/*.so || echo "::warning::strip on extension .so files failed, skipped"
RUN cd ${INSTALL_PATH}/etc && cp php-fpm.conf.default php-fpm.conf && cp php-fpm.d/www.conf.default php-fpm.d/www.conf
RUN cp php-${PHP_VERSION}/php.ini-production ${INSTALL_PATH}/lib/php.ini
RUN sed -i 's/;date.timezone =/date.timezone = PRC/g' ${INSTALL_PATH}/lib/php.ini
RUN sed -i 's/expose_php = On/expose_php = Off/g' ${INSTALL_PATH}/lib/php.ini
RUN sed -i 's/;pid = run/pid = run/g' ${INSTALL_PATH}/etc/php-fpm.conf
RUN sed -i "s/listen = 127.0.0.1:9000/listen = 0.0.0.0:9000/" ${INSTALL_PATH}/etc/php-fpm.d/www.conf
# Only add an ini entry when the .so actually exists, otherwise php-fpm would fail to start at runtime
RUN ls ${INSTALL_PATH}/lib/php/extensions/*/redis.so >/dev/null 2>&1 \
    && printf '\n[Redis]\nextension = redis.so\n' >> ${INSTALL_PATH}/lib/php.ini \
    || echo "::warning::redis.so not built, ini entry skipped"
RUN ls ${INSTALL_PATH}/lib/php/extensions/*/event.so >/dev/null 2>&1 \
    && printf '\n[Event]\nextension = event.so\n' >> ${INSTALL_PATH}/lib/php.ini \
    || echo "::warning::event.so not built, ini entry skipped"
RUN ls ${INSTALL_PATH}/lib/php/extensions/*/apcu.so >/dev/null 2>&1 \
    && printf '\n[Apcu]\nextension = apcu.so\n' >> ${INSTALL_PATH}/lib/php.ini \
    || echo "::warning::apcu.so not built, ini entry skipped"
RUN ls ${INSTALL_PATH}/lib/php/extensions/*/protobuf.so >/dev/null 2>&1 \
    && printf '\n[Protobuf]\nextension = protobuf.so\n' >> ${INSTALL_PATH}/lib/php.ini \
    || echo "::warning::protobuf.so not built, ini entry skipped"
RUN ls ${INSTALL_PATH}/lib/php/extensions/*/rdkafka.so >/dev/null 2>&1 \
    && printf '\n[Rdkafka]\nextension = rdkafka.so\n' >> ${INSTALL_PATH}/lib/php.ini \
    || echo "::warning::rdkafka.so not built, ini entry skipped"
RUN ls ${INSTALL_PATH}/lib/php/extensions/*/grpc.so >/dev/null 2>&1 \
    && printf '\n[Grpc]\nextension = grpc.so\n' >> ${INSTALL_PATH}/lib/php.ini \
    || echo "::warning::grpc.so not built, ini entry skipped"

# Install Composer
RUN ${INSTALL_PATH}/bin/php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
  && ${INSTALL_PATH}/bin/php -r "if (hash_file('SHA384', 'composer-setup.php') === rtrim(file_get_contents('https://composer.github.io/installer.sig'))) { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;" \
  && ${INSTALL_PATH}/bin/php composer-setup.php \
  && ${INSTALL_PATH}/bin/php -r "unlink('composer-setup.php');" \
  && mv composer.phar /composer

FROM ghcr.io/tikobus/rockylinux:9.8.20260525.0-minimal
RUN microdnf install -y epel-release && microdnf clean all
RUN microdnf --enablerepo=epel install -y curl freetype gmp libevent libxml2 libxslt libpng libjpeg libicu librdkafka libpq libsodium libtidy libzip oniguruma openssl readline zlib && microdnf clean all
ENV INSTALL_PATH=/usr/local/php/8.6
COPY --from=build ${INSTALL_PATH} ${INSTALL_PATH}
COPY --from=build /composer /usr/local/bin/composer
COPY --from=build /pie.phar /usr/local/bin/pie
ENV PATH=${INSTALL_PATH}/bin/:${INSTALL_PATH}/sbin/:$PATH
CMD [ "php-fpm", "-F" ]
