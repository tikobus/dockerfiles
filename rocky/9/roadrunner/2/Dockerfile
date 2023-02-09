FROM rockylinux:9.0.20220720-minimal AS build
RUN microdnf install -y git golang make

# Install RoadRunner
RUN git clone --depth 1 https://github.com/spiral/roadrunner \
  && cd roadrunner \
  && make \
  && make install
COPY .rr.yaml /.rr.yaml
COPY rr /rr

# Install Composer
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
  && php -r "if (hash_file('SHA384', 'composer-setup.php') === rtrim(file_get_contents('https://composer.github.io/installer.sig'))) { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;" \
  && php composer-setup.php \
  && php -r "unlink('composer-setup.php');" \
  && mv composer.phar /composer

FROM ghcr.io/tikobus/php:8.2-rocky
COPY --from=build /rr /usr/local/bin/rr
COPY --from=build /.rr.yaml /etc/.rr.yaml
COPY --from=build /composer /usr/local/bin/composer
ENTRYPOINT ["/usr/local/bin/rr", "serve", "-c", "/etc/.rr.yaml"]
