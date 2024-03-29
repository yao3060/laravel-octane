# Accepted values: 8.1 - 8.0
ARG PHP_VERSION=8.1

ARG COMPOSER_VERSION=2

###########################################
# PHP dependencies
###########################################

FROM composer:${COMPOSER_VERSION} as vendor
WORKDIR /var/www/html
# copy source code to image
COPY ./src/composer.json ./src/composer.lock ./
RUN composer install \
    --no-scripts \
    --no-dev \
    --no-interaction \
    --prefer-dist \
    --ignore-platform-reqs \
    --optimize-autoloader \
    --apcu-autoloader \
    --ansi

###########################################
# php image
###########################################
FROM php:${PHP_VERSION}-cli-buster as php

LABEL maintainer="Seyed Morteza Ebadi <seyed.me720@gmail.com>"

ARG WWWUSER=1000
ARG WWWGROUP=1000
ARG TZ=UTC

# Accepted values: app - horizon
ARG CONTAINER_MODE=app

ARG APP_WITH_HORIZON=false

ENV DEBIAN_FRONTEND=noninteractive \
    TERM=xterm-color \
    CONTAINER_MODE=${CONTAINER_MODE} \
    APP_WITH_HORIZON=${APP_WITH_HORIZON}

WORKDIR /var/www/html

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime \
    && echo $TZ > /etc/timezone

RUN set -eu; \
    apt-get update; \
    apt-get upgrade -yqq; \
    pecl channel-update pecl.php.net; \
    apt-get install -yqq --no-install-recommends \
          apt-utils \
          gnupg \
          gosu \
          git \
          curl \
          libcurl4-openssl-dev \
          ca-certificates \
          supervisor \
          libmemcached-dev \
          libz-dev \
          libbrotli-dev \
          libpq-dev \
          libjpeg-dev \
          libpng-dev \
          libfreetype6-dev \
          libssl-dev \
          libwebp-dev \
          libmcrypt-dev \
          libonig-dev \
          libzip-dev zip unzip \
          libargon2-1 \
          libidn2-0 \
          libpcre2-8-0 \
          libpcre3 \
          libxml2 \
          libzstd1 \
          procps

###########################################
# pdo_mysql
###########################################

RUN docker-php-ext-install pdo_mysql;

###########################################
# zip
###########################################

RUN docker-php-ext-configure zip && docker-php-ext-install zip;

###########################################
# mbstring
###########################################

RUN docker-php-ext-install mbstring;

###########################################
# GD
###########################################

RUN docker-php-ext-configure gd \
            --prefix=/usr \
            --with-jpeg \
            --with-webp \
            --with-freetype \
    && docker-php-ext-install gd;

###########################################
# OPcache
###########################################

ARG INSTALL_OPCACHE=true

RUN if [ ${INSTALL_OPCACHE} = true ]; then \
      docker-php-ext-install opcache; \
    fi

###########################################
# PHP Redis
###########################################

ARG INSTALL_PHPREDIS=true

RUN if [ ${INSTALL_PHPREDIS} = true ]; then \
      pecl install -o -f redis \
      && rm -rf /tmp/pear \
      && docker-php-ext-enable redis; \
    fi

###########################################
# PCNTL
###########################################

ARG INSTALL_PCNTL=true

RUN if [ ${INSTALL_PCNTL} = true ]; then \
      docker-php-ext-install pcntl; \
    fi

###########################################
# BCMath
###########################################

ARG INSTALL_BCMATH=true

RUN if [ ${INSTALL_BCMATH} = true ]; then \
      docker-php-ext-install bcmath; \
  fi

###########################################
# RDKAFKA
###########################################

ARG INSTALL_RDKAFKA=true

RUN if [ ${INSTALL_RDKAFKA} = true ]; then \
      apt-get install -yqq librdkafka-dev \
      && pecl install rdkafka \
      && docker-php-ext-enable rdkafka; \
  fi

###########################################
# Swoole extension
###########################################

ARG INSTALL_SWOOLE=true

RUN set -eu; \
    if [ ${INSTALL_SWOOLE} = true ]; then \
      apt-get install -yqq libc-ares-dev \
      && pecl install -D 'enable-openssl="yes" enable-http2="yes" enable-swoole-curl="yes" enable-mysqlnd="yes" enable-cares="yes"' swoole \
      && docker-php-ext-enable swoole; \
    fi

###########################################################################
# Human Language and Character Encoding Support
###########################################################################

ARG INSTALL_INTL=true

RUN if [ ${INSTALL_INTL} = true ]; then \
      apt-get install -yqq zlib1g-dev libicu-dev g++ \
      && docker-php-ext-configure intl \
      && docker-php-ext-install intl; \
  fi

###########################################
# Memcached
###########################################

ARG INSTALL_MEMCACHED=false

RUN if [ ${INSTALL_MEMCACHED} = true ]; then \
      pecl -q install memcached && docker-php-ext-enable memcached; \
  fi

###########################################
# MySQL Client
###########################################

ARG INSTALL_MYSQL_CLIENT=false

RUN if [ ${INSTALL_MYSQL_CLIENT} = true ]; then \
      apt-get install -yqq default-mysql-client; \
  fi

###########################################
# pdo_pgsql
###########################################

ARG INSTALL_PDO_PGSQL=false

RUN if [ ${INSTALL_PDO_PGSQL} = true ]; then \
      docker-php-ext-install pdo_pgsql; \
  fi

###########################################
# pgsql
###########################################

ARG INSTALL_PGSQL=false

RUN if [ ${INSTALL_PGSQL} = true ]; then \
      docker-php-ext-install pgsql; \
  fi

###########################################
# pgsql client and postgis
###########################################

ARG INSTALL_PG_CLIENT=false
ARG INSTALL_POSTGIS=false

RUN if [ ${INSTALL_PG_CLIENT} = true ]; then \
      . /etc/os-release \
      && echo "deb http://apt.postgresql.org/pub/repos/apt $VERSION_CODENAME-pgdg main" > /etc/apt/sources.list.d/pgdg.list \
      && curl -sL https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add - \
      && apt-get install -yqq postgresql-client-12 postgis; \
      if [ ${INSTALL_POSTGIS} = true ]; then \
        apt-get install -yqq postgis; \
      fi; \
    fi

###########################################
# install node for watch
###########################################

# This really shouldn't be modified, so we aren't advertising the env variable.
# It allows us to globally install chokidar without modifying the Laravel package.json.
ENV NODE_PATH "/home/www-data/.npm-global/lib/node_modules"

RUN curl -sL https://deb.nodesource.com/setup_16.x -o /tmp/nodesource_setup.sh && \
    bash /tmp/nodesource_setup.sh && \
    apt-get install -yqq nodejs npm && \
    mkdir -p "/home/www-data/.npm-global/" && \
    npm config set prefix "/home/www-data/.npm-global/" && \
    npm install -g chokidar

###########################################

RUN groupadd --force -g $WWWGROUP octane \
    && useradd -ms /bin/bash --no-user-group -g $WWWGROUP -u $WWWUSER octane

RUN apt-get clean \
    && docker-php-source delete \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
    && rm /var/log/lastlog /var/log/faillog

COPY ./src .
COPY --from=vendor /var/www/html/vendor vendor

RUN mkdir -p \
  storage/framework/{sessions,views,cache} \
  storage/logs \
  bootstrap/cache \
  && chown -R octane:octane \
  storage \
  bootstrap/cache \
  && chmod -R ug+rwx storage bootstrap/cache

COPY .docker/octane/supervisord.${CONTAINER_MODE}.conf /etc/supervisor/conf.d/supervisord.${CONTAINER_MODE}.conf
COPY .docker/octane/php.ini /usr/local/etc/php/conf.d/octane.ini
COPY .docker/octane/opcache.ini /usr/local/etc/php/conf.d/opcache.ini

COPY .docker/octane/entrypoint.sh /entrypoint.sh
COPY .docker/octane/utilities.sh /utilities.sh

RUN chmod +x /entrypoint.sh
RUN cat /utilities.sh >> ~/.bashrc

EXPOSE 9000

# Allow the user to specify Swoole options via ENV variables.
ENV SWOOLE_MAX_REQUESTS "500"
ENV SWOOLE_TASK_WORKERS "auto"
ENV SWOOLE_WATCH $false
ENV SWOOLE_WORKERS "auto"

RUN echo $SWOOLE_WATCH

ENTRYPOINT ["/entrypoint.sh"]

HEALTHCHECK --start-period=8s --interval=5s --timeout=5s CMD php artisan octane:status || exit 1
