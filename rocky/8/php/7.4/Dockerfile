FROM rockylinux/rockylinux:8.10-minimal AS build
ENV PHP_VERSION 7.4.33
# EL8: devel 仓库的名字是 PowerTools(EL9 叫 devel), libzip-devel 等需要启用
RUN microdnf install -y epel-release
RUN microdnf install -y autoconf automake curl-devel freetype-devel gcc gcc-c++ libicu-devel libjpeg-turbo-devel libpng-devel libxml2-devel libzip-devel make openssl-devel readline-devel sqlite-devel tar zlib-devel
RUN microdnf --enablerepo=epel --enablerepo=devel install -y libsodium-devel oniguruma-devel

# EL8 仓库里的 librdkafka 是 0.11, 而 pecl rdkafka 6.x 需要 librdkafka >= 1.0, 因此源码编译
RUN export VERSION=1.9.2 \
    && curl -fSL -o librdkafka-${VERSION}.tar.gz https://github.com/edenhill/librdkafka/archive/refs/tags/v${VERSION}.tar.gz \
    && tar zxf librdkafka-${VERSION}.tar.gz \
    && cd librdkafka-${VERSION} \
    && ./configure --prefix=/usr/local \
    && make -j4 && make install

ENV PKG_CONFIG_PATH /usr/local/lib/pkgconfig/

RUN curl -fSL -O https://www.php.net/distributions/php-${PHP_VERSION}.tar.gz \
    && tar zxf php-${PHP_VERSION}.tar.gz
RUN microdnf --enablerepo=PowerTools install -y libevent-devel
RUN cd php-${PHP_VERSION} \
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
RUN strip /usr/local/php/lib/php/extensions/no-debug-non-zts-20190902/*.so
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

FROM rockylinux/rockylinux:8.10-minimal
RUN microdnf install -y epel-release && microdnf clean all
RUN microdnf --enablerepo=epel install -y curl freetype libevent libicu libjpeg-turbo libpng libsodium libstdc++ libxml2 libzip oniguruma openssl readline zlib && microdnf clean all
COPY --from=build /usr/local/php /usr/local/php
# 源码编译的 librdkafka 安装在 /usr/local/lib, 需要带入运行镜像并注册动态库路径
COPY --from=build /usr/local/lib /usr/local/lib
RUN echo /usr/local/lib > /etc/ld.so.conf.d/usr-local.conf && ldconfig
COPY --from=build /composer /usr/local/bin/composer
ENV PATH /usr/local/php/bin/:/usr/local/php/sbin/:$PATH
CMD [ "php-fpm", "-F" ]
