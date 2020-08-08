FROM php:7.1-fpm-alpine

LABEL maintainer="shanezhiu <shanezhiu@gmail.com>"

# 换国内源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apk/repositories

# memcached拓展版本

ENV MEMCACHED_VERSION 3.1.5

# redis 拓展版本

ENV REDIS_VERSION 5.3.1

# memcache 拓展版本
ENV MEMCACHE_VERSION 4.0.5.2

# imagick 拓展版本

ENV IMAGICK_VERSION 3.4.4

# apcu 拓展版本

ENV APCU_VERSION 5.1.18

# GD库依赖
ENV GD_DEPENS \
            freetype-dev \
            libjpeg-turbo-dev \
            libpng-dev \
            libwebp-dev \
            giflib-dev

RUN set -eux; \
    # 安装gd库
    apk add --no-cache --virtual .run-deps \
        $PHPIZE_DEPS \
        # GD依赖库
        $GD_DEPENS \
        # imagemagick依赖
        imagemagick-dev \
        # memcached 依赖
        libmemcached-dev \
        # openssl-dev
        openssl-dev \
        # gettext
        gettext \
    ; \
    \
    # 安装 pdo_mysql,mysqli,, bcmath, opcached, sockets,pcntl,calendar,sysvmsg,sysvsem, sysvshm拓展
    \
    docker-php-ext-install pdo_mysql mysqli bcmath opcache sockets pcntl calendar sysvmsg sysvsem sysvshm \
            && docker-php-ext-configure gd --with-freetype-dir --with-jpeg-dir --with-png-dir --with-webp-dir \
            && docker-php-ext-install -j$(nproc) gd \
    ; \
    \
    # 安装apcu拓展
    \
    pecl install apcu-$APCU_VERSION \
        && docker-php-ext-enable apcu \
    ; \
    \
    # 安装redis拓展
    \
    pecl install redis-$REDIS_VERSION \
        && docker-php-ext-enable redis \
    ; \
    \
    # 安装memcached 拓展
    \
    pecl install memcached-$MEMCACHED_VERSION \
        && docker-php-ext-enable memcached \
    ; \
    \
    # 安装imagemagick拓展
    \
    pecl install imagick-$IMAGICK_VERSION \
        && docker-php-ext-enable imagick \
    ; \
    \
    runDeps="$( \
        scanelf --needed --nobanner --format '%n#p' --recursive /usr/local \
            | tr ',' '\n' \
            | sort -u \
            | awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
    )"; \
    apk add --no-cache $runDeps; \
    \
    # 删除依赖，减少image大小
    \
    apk del --no-network .run-deps; \
    php -m

# 使用开发环境php.ini

RUN mv "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini"

CMD ["php-fpm", "-F"]