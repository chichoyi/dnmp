#
# NOTE: THIS DOCKERFILE IS GENERATED VIA "update.sh"
#
# PLEASE DO NOT EDIT IT DIRECTLY.
#

FROM php:7.2.2-fpm

ENV PHPREDIS_VERSION 3.1.3

#for redis and mysql
RUN curl -L -o /tmp/redis.tar.gz https://github.com/phpredis/phpredis/archive/$PHPREDIS_VERSION.tar.gz \
    && tar xfz /tmp/redis.tar.gz \
    && rm -r /tmp/redis.tar.gz \
    && mkdir -p /usr/src/php/ext \
    && mv phpredis-$PHPREDIS_VERSION /usr/src/php/ext/redis \
    && docker-php-ext-install redis \
        pdo pdo_mysql \
        bcmath

#for gd
RUN apt-get update && apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
    && docker-php-ext-install -j$(nproc) iconv \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd

## for grpc extention
#RUN  apt-get install -y \
#        libmemcached-dev zlib1g-dev \
#    && pecl install grpc \
#    && pecl install protobuf \
#    && rm -rf /usr/src/php

# for zip
RUN apt-get update \
    && apt-get install -y --no-install-recommends libzip-dev \
    && rm -r /var/lib/apt/lists/* \
    && docker-php-ext-install -j$(nproc) zip

#for ffmpeg -- aliuyun 没有ffpmeg
#RUN echo "deb [check-valid-until=no] http://archive.debian.org/debian jessie-backports main" > /etc/apt/sources.list.d/jessie-backports.list
#RUN sed -i '/deb http:\/\/deb.debian.org\/debian jessie-updates main/d' /etc/apt/sources.list
#RUN apt-get -o Acquire::Check-Valid-Until=false update
#RUN apt-get -y --force-yes install yasm ffmpeg


##for rabbitmq
#RUN apt-get -qq update && apt-get -qq -y install  \
#         automake \
#         cmake \
#         g++ \
#         git \
#         libicu-dev \
#         libmagickwand-dev \
#         librabbitmq-dev \
#         libreadline-dev \
#         pkg-config \
#         ssh-client \
#         supervisor \
#    && docker-php-ext-install \
#         sockets \
#         intl \
#    && git clone git://github.com/alanxz/rabbitmq-c.git \
#    && cd rabbitmq-c \
#    && mkdir build && cd build \
#    && cmake -DENABLE_SSL_SUPPORT=OFF .. \
#    && cmake --build . --target install  \
#    && pecl install amqp imagick xdebug igbinary \
#    && docker-php-ext-enable amqp imagick xdebug igbinary \
#    && version=$(php -r "echo PHP_MAJOR_VERSION.PHP_MINOR_VERSION;") \
#    && curl -A "Docker" -o /tmp/blackfire-probe.tar.gz -D - -L -s https://blackfire.io/api/v1/releases/probe/php/linux/amd64/$version \
#    && mkdir -p /tmp/blackfire \
#    && tar zxpf /tmp/blackfire-probe.tar.gz -C /tmp/blackfire \
#    && mv /tmp/blackfire/blackfire-*.so $(php -r "echo ini_get('extension_dir');")/blackfire.so \
#    && printf "extension=blackfire.so\nblackfire.agent_socket=tcp://blackfire:8707\n" > $PHP_INI_DIR/conf.d/blackfire.ini \
#    && curl -A "Docker" -L https://blackfire.io/api/v1/releases/client/linux_static/amd64 | tar zxp -C /tmp/blackfire \
#    && mv /tmp/blackfire/blackfire /usr/bin/blackfire \
#    && rm -rf /tmp/blackfire /tmp/blackfire-probe.tar.gz \
#    && rm -rf /var/lib/apt/lists/*