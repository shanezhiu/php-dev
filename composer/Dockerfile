FROM composer:1.9

LABEL maintainer="shanezhiu <shanezhiu@gmail.com>"

# swoole 版本

ENV SWOOLE_VERSION 4.5.2
ENV SWOOLE_SOURCE_DIR /tmp/swoole-src-$SWOOLE_VERSION
ENV SWOOLE_URL https://github.com/swoole/swoole-src/archive/v$SWOOLE_VERSION.tar.gz

# redis 拓展版本

ENV REDIS_VERSION 5.3.1

# 换国内源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apk/repositories


# 安装所需要的拓展
RUN set -eux; \
        # 安装gd库
        apk add --no-cache --virtual .run-deps \
            $PHPIZE_DEPS \
            # intl所需库 
            icu-dev \
            # openssl-dev
            openssl-dev \
            # libbson
            libbson-dev \
        ; \
        \
        # 安装 pdo_mysql bcmath, opcached, sockets,pcntl,calendar,sysvmsg,sysvsem, sysvshm拓展
        \
        docker-php-ext-install pdo_mysql pcntl intl bcmath sockets\
        ; \
        \
        # 安装redis拓展
        \
        pecl install redis-$REDIS_VERSION \
            && docker-php-ext-enable redis \
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
        php -m

# composer源，使用阿里云
RUN composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
# 取消全局配置
# composer config -g --unset repos.packagist
