FROM    composer:2.4 as composer
FROM    php:7.3-fpm-alpine

COPY    --from=composer /usr/bin/composer /usr/local/bin/composer

# Removing APKINDEX warnings
RUN     rm -rf /var/cache/apk/* && \
        rm -rf /tmp/*
RUN     apk update

# Native libs and building dependencies
# su-exec > gosu (10kb instead of 1.8MB)
RUN     apk add --update --no-cache \
        vim \
        git \
        unzip \
        make \
        nodejs \
        yarn \
        zlib-dev \
        libzip-dev \
        ca-certificates \
        && apk add --no-cache --virtual .build-deps \
            $PHPIZE_DEPS \
            curl \
            icu-dev \
        && docker-php-ext-install \
            zip \
            pdo_mysql \
        && yes | pecl install xdebug \
        && docker-php-ext-enable xdebug \
        && apk add --no-cache su-exec \
        && addgroup bar \
        && adduser -D -h /home -s /bin/sh -G bar foo \
        && apk del .build-deps

# intl ? 
RUN apk add icu-dev 
# RUN docker-php-ext-install mysqli pdo pdo_mysql
RUN docker-php-ext-configure intl && docker-php-ext-install intl


# Images libraries
RUN apk add --no-cache freetype libpng libjpeg-turbo freetype-dev libpng-dev libjpeg-turbo-dev && \
  docker-php-ext-configure gd \
    --with-gd \
    --with-freetype-dir=/usr/include/ \
    --with-png-dir=/usr/include/ \
    --with-jpeg-dir=/usr/include/ && \
  NPROC=$(grep -c ^processor /proc/cpuinfo 2>/dev/null || 1) && \
  docker-php-ext-install -j${NPROC} gd && \
  apk del --no-cache freetype-dev libpng-dev libjpeg-turbo-dev

# Blackfire

#ENV current_os=alpine
#RUN version=$(php -r "echo PHP_MAJOR_VERSION.PHP_MINOR_VERSION;") \
    #&& curl -A "Docker" -o /tmp/blackfire-probe.tar.gz -D - -L -s https://blackfire.io/api/v1/releases/probe/php/$current_os/amd64/$version \
    #&& mkdir -p /tmp/blackfire \
    #&& tar zxpf /tmp/blackfire-probe.tar.gz -C /tmp/blackfire \
    #&& mv /tmp/blackfire/blackfire-*.so $(php -r "echo ini_get('extension_dir');")/blackfire.so \
    #&& printf "extension=blackfire.so\nblackfire.agent_socket=tcp://blackfire:8707\n" > $PHP_INI_DIR/conf.d/blackfire.ini \
    #&& rm -rf /tmp/blackfire /tmp/blackfire-probe.tar.gz

# Sonde Blackfire 
RUN version=$(php -r "echo PHP_MAJOR_VERSION.PHP_MINOR_VERSION;") \
    && architecture=$(uname -m) \
    && curl -A "Docker" -o /tmp/blackfire-probe.tar.gz -D - -L -s https://blackfire.io/api/v1/releases/probe/php/alpine/$architecture/$version \
    && mkdir -p /tmp/blackfire \
    && tar zxpf /tmp/blackfire-probe.tar.gz -C /tmp/blackfire \
    && mv /tmp/blackfire/blackfire-*.so $(php -r "echo ini_get ('extension_dir');")/blackfire.so \
    && printf "extension=blackfire.so\nblackfire.agent_socket=tcp://blackfire:8307\n" > $PHP_INI_DIR/conf.d/blackfire.ini \
    && rm -rf /tmp/blackfire /tmp/blackfire-probe.tar.gz


# PHP config
COPY    conf.d/php.ini /usr/local/etc/php
COPY    conf.d/symfony.ini /usr/local/etc/php/conf.d

# Composer 
RUN php -r "readfile('http://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer


# other 
RUN echo 'memory_limit = 2048M' >> /usr/local/etc/php/conf.d/docker-php-memlimit.ini;


# require  ext-mongodb ^1.14.0
RUN apk update \
    && apk add --no-cache \
    ${PHPIZE_DEPS} \
    && pecl install mongodb \
    && docker-php-ext-enable \
    mongodb \
    && apk del