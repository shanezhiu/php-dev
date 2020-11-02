FROM shanezhiu/php:7.1

LABEL maintainer="shanezhiu <shanezhiu@gmail.com>"

# 安装scws分词拓展

ENV SCWS_VERSION 1.2.3
ENV SCWS_SOURCE_DIR /tmp/scws-$SCWS_VERSION/phpext
ENV SCWS_URL http://www.xunsearch.com/scws/down/scws-$SCWS_VERSION.tar.bz2

RUN set -eux; \
	apk add --no-cache --virtual .run-deps \
	$PHPIZE_DEPS \
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
	\
	# php-fpm alpine 镜像里imagick-dev formats为0,必须安装imagemagick
        # bug:https://github.com/docker-library/php/issues/105#issuecomment-249716758
	# bug:https://gitlab.alpinelinux.org/alpine/aports/-/issues/11522 
	# imagick queryformats 为空,
	# NoDecodeDelegateForThisImageFormat `PNG' @ error/constitute.c/ReadImage/562
	# fix: apk add imagemagick
	\
	apk add imagemagick; \
	php -m
