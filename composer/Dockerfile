FROM composer:1.9

LABEL maintainer="shanezhiu <shanezhiu@gmail.com>"


# 换国内源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apk/repositories


# 安装所需要的拓展
RUN set -eux; \
        # 安装gd库
        apk add --no-cache --virtual .run-deps \
            $PHPIZE_DEPS \
            # intl所需库 
            icu-dev; \
        \
        docker-php-ext-install intl bcmath\
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