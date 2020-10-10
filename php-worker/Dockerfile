FROM php:7.4.8-cli-alpine

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


# 安装scws分词拓展

ENV SCWS_VERSION 1.2.3
ENV SCWS_SOURCE_DIR /tmp/scws-$SCWS_VERSION/phpext
ENV SCWS_URL http://www.xunsearch.com/scws/down/scws-$SCWS_VERSION.tar.bz2

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
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apk/repositories

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
        # 安装 pdo_mysql bcmath, opcached, sockets,pcntl,calendar,sysvmsg,sysvsem, sysvshm拓展
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
        # 安装scws拓展	
        \
        curl --noproxy "*" --progress-bar -v -fsSL -o /tmp/scws.tar.bz2 "$SCWS_URL"; \
        cd /tmp && tar -xvf scws.tar.bz2 -C /tmp; \
        \
        # 先安装c库
        \
        cd $SCWS_SOURCE_DIR && cd ../ && ./configure && make install \
        ; \
        \
        cd ${SCWS_SOURCE_DIR}; \
        phpize && ./configure; \
        make -j$(nproc) && make install; \
        make clean \
        ;\
        \
        # 生成拓展ini
        \
        touch /usr/local/etc/php/conf.d/scws.ini; \
        echo 'extension = scws.so' > /usr/local/etc/php/conf.d/scws.ini; \
        rm -rf ${SCWS_SOURCE_DIR} /tmp/scws.tar.gz \
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

CMD ["php", "-v"]
