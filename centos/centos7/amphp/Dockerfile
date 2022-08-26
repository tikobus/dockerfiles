FROM ghcr.io/tikobus/php:8.1 AS build
WORKDIR /usr/local
RUN export VERSION=2.4.1 && curl -o bin/composer https://getcomposer.org/download/${VERSION}/composer.phar && chmod +x bin/composer
RUN mkdir -p /www/amphp && cd /www/amphp \
  && composer require amphp/amp:v2.6.2 \
  && composer require amphp/http-client \
  && composer require amphp/http-server \
  && composer require amphp/mysql \
  && composer require amphp/socket \
  && composer require amphp/redis \
  && composer require amphp/log \
  && composer require amphp/websocket \
  && composer require amphp/process \
  && composer require amphp/parser

COPY httpServer.php /www/amphp/httpServer.php

FROM ghcr.io/tikobus/php:8.1
WORKDIR /www/amphp/
COPY --from=build /usr/local/bin/composer /usr/local/bin/composer
COPY --from=build /www /www
