FROM php:7.1-fpm-alpine

LABEL maintainer="shanezhiu <shanezhiu@gmail.com>"

# 换国内源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# memcached拓展版本

ENV MEMCACHED_VERSION 3.1.5

# redis 拓展版本

ENV REDIS_VERSION 5.3.2

# memcache 拓展版本
ENV MEMCACHE_VERSION 4.0.5.2

# imagick 拓展版本

ENV IMAGICK_VERSION 3.4.4

# mongodb

ENV MONGO_VERSION 1.14.0
ENV MONGO_URL https://github.com/mongodb/mongo-c-driver/archive/$MONGO_VERSION.tar.gz
ENV MONGO_DIR  /tmp/mongo-c-driver-$MONGO_VERSION

# mongodb 拓展版本

ENV MONGO_PHP_VERSION 1.8.1

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
        # cmake
        cmake \
    ; \
    \
    # 安装mongo-c-driver
    # 这个版本的aplpine有个提交记录：“Upstream has switched to a nonfree license＂
    # 所以需要自己手动安装mongo-c-driver
    # link: https://git.alpinelinux.org/aports/commit/testing?id=8a901de31fa055ed591d487e12f8bb9ffcc0df21
    # link: https://gitlab.alpinelinux.org/alpine/aports/-/issues/10221#note-4
    # link: https://github.com/instrumentisto/coturn-docker-image/commit/9bcdedabefb33b41e2047fa12bca46eb20c4fc98
    curl --noproxy "*" --progress-bar -v -fsSL -o /tmp/mongo.tar.gz "$MONGO_URL"; \
    cd /tmp && tar -zxvf mongo.tar.gz -C /tmp; \
    mkdir -p /tmp/build/mongo-c-driver/ && cd /tmp/build/mongo-c-driver/; \
    cmake -DCMAKE_BUILD_TYPE=Release \
          -DCMAKE_INSTALL_PREFIX=/usr \
          -DCMAKE_INSTALL_LIBDIR=lib \
          -DENABLE_BSON:STRING=ON \
          -DENABLE_MONGOC:BOOL=ON \
          -DENABLE_SSL:STRING=OPENSSL \
          -DENABLE_AUTOMATIC_INIT_AND_CLEANUP:BOOL=OFF \
          -DENABLE_MAN_PAGES:BOOL=OFF \
          -DENABLE_TESTS:BOOL=ON \
          -DENABLE_EXAMPLES:BOOL=OFF \
          -DCMAKE_SKIP_RPATH=ON \
        $MONGO_DIR \
    && make \
    # Check mongo-c-driver build
    && MONGOC_TEST_SKIP_MOCK=on \
        MONGOC_TEST_SKIP_SLOW=on \
        MONGOC_TEST_SKIP_LIVE=on \
        make check \
    \
    # Install mongo-c-driver
    && make install \
    ; \
    \
    # 清除
    \
    rm -rf /tmp/build/mongo-c-driver/ && rm -f /tmp/mongo.tar.gz \
    ; \
    \
    # 更新pecl协议
    \
    pecl channel-update pecl.php.net \
    ; \
    \
    # 安装 pdo_mysql,mysqli,, bcmath, opcached, sockets,pcntl,calendar,sysvmsg,sysvsem, sysvshm拓展
    \
    docker-php-ext-install pdo_mysql mysqli bcmath opcache sockets pcntl calendar sysvmsg sysvsem sysvshm \
            && docker-php-ext-configure gd --with-freetype-dir --with-jpeg-dir --with-png-dir --with-webp-dir \
            && docker-php-ext-install -j$(nproc) gd \
    ; \
    \
    # 安装redis拓展
    \
    pecl install redis-$REDIS_VERSION \
        && docker-php-ext-enable redis \
    ; \
    \
    #安装　mongodb拓展
    \
    pecl install mongodb-$MONGO_PHP_VERSION \
        && docker-php-ext-enable mongodb \
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
