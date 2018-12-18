#
# NOTE: THIS DOCKERFILE IS GENERATED VIA "update.sh"
#
# PLEASE DO NOT EDIT IT DIRECTLY.
#

FROM php:7.2.2-fpm

ENV PHPREDIS_VERSION 3.1.3

RUN curl -L -o /tmp/redis.tar.gz https://github.com/phpredis/phpredis/archive/$PHPREDIS_VERSION.tar.gz \
    && tar xfz /tmp/redis.tar.gz \
    && rm -r /tmp/redis.tar.gz \
    && mkdir -p /usr/src/php/ext \
    && mv phpredis-$PHPREDIS_VERSION /usr/src/php/ext/redis \
    && docker-php-ext-install redis \
    && docker-php-ext-install pdo pdo_mysql \
#    && curl -l https://pecl.php.net/get/amqp-1.9.3.tgz | tar xvz -C /usr/src/php/ext/amqp --strip 1 \
#    && /usr/src/php-available-exts \
#    && cp /usr/src/rabbitmq-c/librabbitmq/amqp_ssl_socket.h /usr/src/php/ext/amqp/ \
#    && docker-php-ext-configure amqp --with-librabbitmq-dir=/usr/local/rabbitmq-c \
#    && docker-php-ext-install amqp \
#    && php -m | grep amqp \
    && rm -rf /usr/src/php