FROM php:8.3.0-fpm-alpine3.17

WORKDIR /var/www/html

RUN apk --no-cache add nginx

# ------------------------ Common PHP Dependencies ------------------------
RUN apk --no-cache add su-exec bash sed git openssh-client icu shadow procps \
        freetype libpng libjpeg-turbo libzip-dev ghostscript imagemagick \
        jpegoptim optipng pngquant gifsicle libldap \
        libpq less \
    # build-deps
    && apk add --no-cache --virtual .build-deps $PHPIZE_DEPS \
        freetype-dev libpng-dev libjpeg-turbo-dev \
        icu-dev libedit-dev libxml2-dev \
        imagemagick-dev openldap-dev oniguruma-dev \
        postgresql-dev \
        linux-headers \
    # php-ext
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && export CFLAGS="$PHP_CFLAGS" CPPFLAGS="$PHP_CPPFLAGS" LDFLAGS="$PHP_LDFLAGS" \
    && docker-php-ext-install -j$(nproc) \
        bcmath \
        calendar \
        exif \
        gd \
        intl \
        ldap \
        mbstring \
        opcache \
        pcntl \
        pdo \
        pdo_mysql \
        pdo_pgsql \
        soap \
        xml \
        zip \
        sockets \
        mysqli \
        ftp \
    && pecl install redis \
    && mkdir /tmp/imagick && cd /tmp/imagick \
    && curl -L -o /tmp/imagick.tar.gz https://github.com/Imagick/imagick/archive/7088edc353f53c4bc644573a79cdcd67a726ae16.tar.gz \
    && tar --strip-components=1 -xf /tmp/imagick.tar.gz \
    && phpize \
    && ./configure \
    && make \
    && make install \
    && echo "extension=imagick.so" > /usr/local/etc/php/conf.d/ext-imagick.ini \
    && docker-php-ext-enable redis \
    && cp "/usr/local/etc/php/php.ini-production" "/usr/local/etc/php/php.ini" \
    # composer
    && curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer \
    && curl -sS https://getcomposer.org/installer | php -- --1 --install-dir=/usr/local/bin --filename=composer1 \
    # cleanup
    && apk del .build-deps \
    && rm -rf /var/cache/apk/* /tmp/*

# ------------------------ xdebug ------------------------
ARG XDEBUG_ENABLE=false
ARG REMOTE_XDEBUG_HOST=0.0.0.0
ARG REMOTE_XDEBUG_PORT=9999
RUN if [ "$XDEBUG_ENABLE" = "true" ]; then \
	pecl install xdebug-3.0.0; \
    docker-php-ext-enable xdebug; \
    echo "xdebug.remote_enable=1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
    echo "xdebug.remote_autostart=1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
    echo "xdebug.remote_host=${REMOTE_XDEBUG_HOST}" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
    echo "xdebug.remote_port=${REMOTE_XDEBUG_PORT}" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
    echo "xdebug.remote_handler=dbgp" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
    echo "xdebug.remote_connect_back=On" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
    echo "xdebug.idekey=x_debug" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
	mv /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini.disabled; \
	fi

# ------------------------ start fpm/nginx ------------------------

COPY entrypoint.sh /etc/entrypoint.sh
RUN chmod +x /etc/entrypoint.sh
RUN chown -R www-data:www-data /var/www/html /run /var/lib/nginx /var/log/nginx
RUN touch /var/run/nginx.pid && \
    chown -R www-data:www-data /var/run/nginx.pid


# Adding the opcache configuration into the wrong directory intentionally.
# This is enabled at runtime
ADD opcache.ini /usr/local/etc/php/opcache_disabled.ini


# PHP-FPM Status
RUN set -xe && echo "pm.status_path = /status" >> /usr/local/etc/php-fpm.d/zz-docker.conf && \
    echo "ping.path = /ping" >> /usr/local/etc/php-fpm.d/zz-docker.conf && \
    echo "ping.response = pong" >> /usr/local/etc/php-fpm.d/zz-docker.conf
    
RUN rm -rf /var/cache/apk/* && \
        rm -rf /tmp/*

USER www-data

EXPOSE 80

ENTRYPOINT ["/etc/entrypoint.sh"]
