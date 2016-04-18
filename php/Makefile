SRC=/usr/src/php
PHP_INI_DIR=/usr/local/etc/php

# version setting
GPG_KEYS=1A4E8B7277C42E53DBA9C7B9BCAA30EA9C0D5763
PHP_VERSION=7.0.5
PHP_FILENAME=php-7.0.5.tar.xz
PHP_SHA256=c41f1a03c24119c0dd9b741cdb67880486e64349fc33527767f6dc28d3803abb

# step0. prepare - phpize & runtime deps
install-base: 
	apt-get update && apt-get install -y \
	autoconf \
	file \
	g++ \
	gcc \
	libc-dev \
	make \
	pkg-config \
	re2c \
	ca-certificates \
	curl \
	libedit2 \
	libsqlite3-0 \
	libxml2 \
	git \
	--no-install-recommends && rm -r /var/lib/apt/lists/*

# step1. download & extract
download: 
	set -xe \
	&& curl -fSL "http://php.net/get/$(PHP_FILENAME)/from/this/mirror" -o "$(PHP_FILENAME)" \
	&& echo "$(PHP_SHA256) *$(PHP_FILENAME)" | sha256sum -c - \
	&& curl -fSL "http://php.net/get/$(PHP_FILENAME).asc/from/this/mirror" -o "$(PHP_FILENAME).asc" \
	&& export GNUPGHOME="$$(mktemp -d)" \
	&& for key in $(GPG_KEYS); do \
		gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$$key"; \
		done \
	&& gpg --batch --verify "$(PHP_FILENAME).asc" "$(PHP_FILENAME)" \
	&& rm -r "$$GNUPGHOME" "$(PHP_FILENAME).asc" \
	&& mkdir -p $(SRC) \
	&& tar -xf "$(PHP_FILENAME)" -C $(SRC) --strip-components=1 \
	&& rm "$(PHP_FILENAME)"

install-deps:
	set -xe \
	&& buildDeps=" \
		$(PHP_EXTRA_BUILD_DEPS) \
		libcurl4-openssl-dev \
		libedit-dev \
		libsqlite3-dev \
		libssl-dev \
		libxml2-dev \
		libfreetype6-dev \
		libjpeg-dev \
		libmcrypt-dev \
		libpng12-dev \
		librabbitmq-dev \
		libyaml-dev \
		xz-utils \
		" \
	&& apt-get update \
	&& apt-get install -y $$buildDeps --no-install-recommends && rm -rf /var/lib/apt/lists/*

# step3. build
install-php:
	set -xe \
	&& mkdir -p $(PHP_INI_DIR)/conf.d \
	&& cd $(SRC) \
	&& ./configure \
		--with-config-file-path="$(PHP_INI_DIR)" \
		--with-config-file-scan-dir="$(PHP_INI_DIR)/conf.d" \
		--enable-fpm --with-fpm-user=www-data --with-fpm-group=www-data \
		--disable-cgi \
		--enable-mysqlnd \
		--enable-mbstring \
		--with-curl \
		--with-libedit \
		--with-openssl \
		--with-zlib \
	&& make -j"$$(nproc)" \
	&& make install \
	&& { find /usr/local/bin /usr/local/sbin -type f -executable -exec strip --strip-all '{}' + || true; } \
	&& make clean \
	&& runDeps="$$( \
		scanelf --needed --nobanner --recursive /usr/local \
			| awk '{ gsub(/,/, "\nso:", $$2); print "so:" $$2 }' \
			| sort -u \
			| xargs -r apk info --installed \
			| sort -u \
		)" \
	&& apk add --virtual .php-rundeps $$runDeps \
	&& apk del .build-deps \
	&& cp php-ext-* /usr/local/bin

# step4. add ext
install-ext:
	set -xe \
        && apt-get install -y $$buildDeps --no-install-recommends && rm -rf /var/lib/apt/lists/* \
	&& php-ext-install -j"$$(nproc)" mcrypt bcmath soap pcntl zip sockets pdo_mysql \
	&& php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ --with-png-dir=/usr \
	&& php-ext-install -j"$$(nproc)" gd

#
install-phalcon:

