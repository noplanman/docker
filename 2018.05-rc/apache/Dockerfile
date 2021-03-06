# DO NOT EDIT: created by update.sh from Dockerfile-debian.template
FROM php:7.1-apache
LABEL maintainer="Philipp Holzer <admin@philipp.info>"

# entrypoint.sh and cron.sh dependencies
RUN set -ex; \
    \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        rsync \
        bzip2 \
        git \
# For mail() support
        sendmail \
    ; \
    rm -rf /var/lib/apt/lists/*;

# install the PHP extensions we need
# see https://friendi.ca/resources/requirements/
RUN set -ex; \
    \
    savedAptMark="$(apt-mark showmanual)"; \
    \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        libxml2-dev \
        mysql-client \
        bash \
        autoconf \
        g++ \
        make \
        openssl \
        libssl-dev \
        libpng-dev \
        libjpeg62-turbo-dev \
        libtool \
        libmcrypt4 \
        libmcrypt-dev \
        imagemagick \
        libmagick++-dev \
        libgraphicsmagick1-dev \
        libfreetype6 \
        libfreetype6-dev \
        librsvg2-2 \
        libcurl4-openssl-dev \
        curl \
    ; \
    \
    debMultiarch="$(dpkg-architecture --query DEB_BUILD_MULTIARCH)"; \
    pecl install imagick-3.4.3; \
        docker-php-ext-enable imagick; \
        pecl clear-cache \
    ; \
    docker-php-ext-configure gd \
        --with-gd \
        --enable-gd-native-ttf \
        --with-freetype-dir=/usr/include/ \
        --with-png-dir=/usr/include/ \
        --with-jpeg-dir=/usr/include/ \
    ; \
    docker-php-ext-install -j 4 curl pdo pdo_mysql xml gd zip opcache mbstring posix ctype json iconv mcrypt \
    ; \
# reset apt-mark's "manual" list so that "purge --auto-remove" will remove all build dependencies
    apt-mark auto '.*' > /dev/null; \
    apt-mark manual $savedAptMark; \
    ldd "$(php -r 'echo ini_get("extension_dir");')"/*.so \
      | awk '/=>/ { print $3 }' \
      | sort -u \
      | xargs -r dpkg-query -S \
      | cut -d: -f1 \
      | sort -u \
      | xargs -rt apt-mark manual; \
    \
    apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
    rm -rf /var/lib/apt/lists/*

RUN chown -R www-data:root /var/www; \
    chmod -R g=u /var/www

VOLUME /var/www/html

RUN a2enmod rewrite remoteip ;\
    {\
     echo RemoteIPHeader X-Real-IP ;\
     echo RemoteIPTrustedProxy 10.0.0.0/8 ;\
     echo RemoteIPTrustedProxy 172.16.0.0/12 ;\
     echo RemoteIPTrustedProxy 192.168.0.0/16 ;\
    } > /etc/apache2/conf-available/remoteip.conf;\
    a2enconf remoteip

RUN {\
      echo sendmail_path = "/usr/sbin/sendmail -t -i" ;\
    } > /usr/local/etc/php/conf.d/sendmail.ini;

ENV AUTOINSTALL false

ENV FRIENDICA_VERSION 2018.05-rc
ENV FRIENDICA_ADDONS 2018.05-rc


COPY bin/* /usr/local/bin/
COPY config/* /usr/src/config/
COPY *.sh /
RUN chmod +x /*.sh
RUN chmod +x /usr/local/bin/*

ENTRYPOINT ["/entrypoint.sh"]
CMD ["apache2-foreground"]
