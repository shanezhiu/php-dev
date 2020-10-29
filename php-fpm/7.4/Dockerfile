FROM php:7.4.8-fpm-alpine

LABEL maintainer="shanezhiu <shanezhiu@gmail.com>"


# php 容器配置文件目录
#ENV PHP_INI_DIR /usr/local/etc/php

# memcached拓展版本

ENV MEMCACHED_VERSION 3.1.5

# redis 拓展版本

ENV REDIS_VERSION 5.3.1

# imagick 拓展版本

ENV IMAGICK_VERSION 3.4.4

# swoole 版本

ENV SWOOLE_VERSION 4.5.2
ENV SWOOLE_SOURCE_DIR /tmp/swoole-src-$SWOOLE_VERSION
ENV SWOOLE_URL https://github.com/swoole/swoole-src/archive/v$SWOOLE_VERSION.tar.gz

# mongodb 拓展版本

ENV MONGO_VERSION 1.8.0

# GD库依赖
ENV GD_DEPENS \
            freetype-dev \
            libjpeg-turbo-dev \
            libpng-dev \
            libwebp-dev \
            giflib-dev

# 换国内源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# 安装所需要的拓展
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
            # libbson
            libbson-dev \
            # mongo-c-driver-dev
            mongo-c-driver-dev \
        ; \
        \
        # 安装 pdo_mysql bcmath, opcache, sockets,pcntl,calendar,sysvmsg,sysvsem, sysvshm拓展
        \
        docker-php-ext-install pdo_mysql bcmath opcache sockets pcntl calendar sysvmsg sysvsem sysvshm \
               && docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp \
               && docker-php-ext-install -j$(nproc) gd \
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
        #安装　mongodb拓展
        \
        pecl install mongodb-$MONGO_VERSION \
            && docker-php-ext-enable mongodb \
        ; \
        \
        # 安装swoole拓展
        curl --noproxy "*" --progress-bar -v -fsSL -o /tmp/swoole.tar.gz "$SWOOLE_URL"; \
        cd /tmp && tar -zxvf swoole.tar.gz -C /tmp; \
        cd $SWOOLE_SOURCE_DIR; \
        phpize && ./configure --enable-openssl --enable-sockets --enable-http2 --enable-mysqlnd; \
        make -j$(nproc) && make install; \
        make clean \
        ; \
        \
        # 生成拓展ini
        \
        touch /usr/local/etc/php/conf.d/swoole.ini; \
        echo 'extension=swoole.so' > /usr/local/etc/php/conf.d/swoole.ini; \
        rm -rf $SWOOLE_SOURCE_DIR /tmp/swoole.tar.gz \
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
	\
	# php-fpm alpine 镜像里imagick-dev formats为0,必须安装imagemagick
        # bug:https://github.com/docker-library/php/issues/105#issuecomment-249716758
	# bug:https://gitlab.alpinelinux.org/alpine/aports/-/issues/11522 
	# imagick queryformats 为空,
	# NoDecodeDelegateForThisImageFormat `PNG' @ error/constitute.c/ReadImage/562
	# fix: apk add imagemagick
        \
	apk add imagemagick; \
	\
        php -m

# 使用开发环境php.ini

RUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"

# 指定用户，如果不指定用户，默认使用root
USER 82:82

CMD ["php-fpm", "-F"]
