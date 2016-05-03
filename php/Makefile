# PATH
SRC=/usr/src/php
PHP_INI_DIR=/usr/local/etc/php

# version setting
GPG_KEYS=1A4E8B7277C42E53DBA9C7B9BCAA30EA9C0D5763
PHP_VERSION=7.0.6
PHP_FILENAME=php-7.0.6.tar.xz
PHP_SHA256=1b237a9455e5476a425dbb9d99966bad68107747c601958cb9558a7fb49ab419

# dep
buildDeps=" \
		  $(PHP_EXTRA_BUILD_DEPS)  \
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
		  xz-utils"

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

# install deps
install-deps:
	set -xe \
	&& apt-get update \
	&& apt-get install -y $(buildDeps) --no-install-recommends && rm -rf /var/lib/apt/lists/*

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
	&& cp $(SRC)/sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm \
	&& chmod +x /etc/init.d/php-fpm \
	&& { find /usr/local/bin /usr/local/sbin -type f -executable -exec strip --strip-all '{}' + || true; } \
	&& make clean \
	&& cp php-ext-* /usr/local/bin

# step4. add ext
install-ext:
	set -xe \ 
	&& apt-get install -y $(buildDeps) --no-install-recommends && rm -rf /var/lib/apt/lists/* \
	&& php-ext-install -j"$$(nproc)" mcrypt bcmath soap pcntl zip sockets pdo_mysql \
	&& php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ --with-png-dir=/usr \
	&& php-ext-install -j"$$(nproc)" gd

# for php 5.6 only, php 7 should manually install it
install-phalcon: 
	set -xe \ 
	&& curl -fSL "https://github.com/phalcon/cphalcon/archive/phalcon-v2.0.10.tar.gz" -o /home/cphalcon.tgz \ 
	&& cd /home \ 
	&& tar -xvf cphalcon.tgz \ 
	&& mv cphalcon-* phalcon \ 
	&& cd phalcon/build && ./install \ 
	&& php-ext-enable phalcon \ 
	&& rm /home/cphalcon.tgz \ 
	&& rm -rf /home/phalcon

# 
uninstall-deps:
	set -xe \
	&& apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false -o APT::AutoRemove::SuggestsImportant=false $(buildDeps)
