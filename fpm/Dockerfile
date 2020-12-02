FROM php:7.4-fpm-alpine

LABEL Maintainer="Ansley Leung" \
      Description="latest PHP7 fpm Docker image. Use `docker-php-ext-install extension_name` to install Extensions." \
      License="MIT License" \
      Version="7.4.13"

# Environments
ENV TZ            Asia/Shanghai

# Set timezone
RUN set -ex && \
    apk update && \
    apk upgrade && \
    apk add --no-cache tzdata && \
    # ln -snf /usr/share/zoneinfo/ /etc/localtime && \
    cp /usr/share/zoneinfo/$TZ /etc/localtime && \
    echo $TZ > /etc/timezone && \
    apk del tzdata && \
    rm -rf /tmp/* /var/cache/apk/*

# Install extensions
# ref to: https://github.com/mlocati/docker-php-extension-installer
RUN set -ex && \
    apk add --no-cache \
        autoconf \
        build-base \
        curl \
        git \
        grep \
        make \
        unzip \
        aspell-dev \
        bzip2-dev \
        curl-dev \
        freetype-dev \
        gettext-dev \
        icu-dev \
        imap-dev \
        krb5-dev \
        recode-dev \
        libbz2 \
        libgcrypt-dev \
        libjpeg-turbo-dev \
        libpng-dev \
        libsodium-dev \
        libstdc++ \
        libwebp-dev \
        libxpm-dev \
        libxml2-dev \
        libxslt-dev \
        libzip-dev \
        mysql-dev \
        net-snmp-dev \
        oniguruma-dev \
        openldap-dev \
        openssh-client \
        openssl-dev \
        postgresql-client \
        postgresql-dev \
        sqlite-dev && \
    docker-php-ext-configure gd --with-freetype --with-jpeg && \
    PHP_OPENSSL=yes docker-php-ext-configure imap --with-kerberos --with-imap-ssl && \
    docker-php-ext-configure ldap --with-libdir=lib/ && \
    docker-php-ext-configure opcache --enable-opcache && \
    docker-php-ext-configure zip && \
    docker-php-ext-install -j$(grep -c ^processor /proc/cpuinfo 2>/dev/null || 1) \
        bcmath \
        bz2 \
        curl \
        exif \
        gd \
        gettext \
        iconv \
        imap \
        intl \
        json \
        ldap \
        mbstring \
        mysqli \
        opcache \
        pcntl \
        pdo \
        pdo_mysql \
        pdo_pgsql \
        pdo_sqlite \
        pspell \
        shmop \
        simplexml \
        snmp \
        soap \
        sockets \
        sysvmsg \
        sysvsem \
        sysvshm \
        xml \
        xsl \
        zip && \
    rm -rf /tmp/* /var/cache/apk/*

# Set recommended opcache settings
# https://secure.php.net/manual/en/opcache.installation.php
RUN { \
            echo 'opcache.memory_consumption=128'; \
            echo 'opcache.interned_strings_buffer=8'; \
            echo 'opcache.max_accelerated_files=4000'; \
            echo 'opcache.revalidate_freq=60'; \
            echo 'opcache.fast_shutdown=1'; \
            echo 'opcache.enable_cli=1'; \
            echo 'opcache.file_cache=/tmp'; \
        } > $PHP_INI_DIR/conf.d/opcache-recommended.ini

# remove PHP version from the X-Powered-By HTTP header
# test: curl -I -H "Accept-Encoding: gzip, deflate" https://www.yourdomain.com
RUN echo 'expose_php = off' > $PHP_INI_DIR/conf.d/hide-header-version.ini

# Composer
RUN set -ex && \
    mkdir -p /usr/local/share/composer && \
    export COMPOSER_ALLOW_SUPERUSER=1 && \
    export COMPOSER_HOME=/usr/local/share/composer && \
    curl -sS https://install.phpcomposer.com/installer | php -- --install-dir=/usr/bin/ --filename=composer

# Install extension using pecl
# pecl install xdebug-beta
RUN set -ex && \
    apk add --no-cache imagemagick-dev \
    libmemcached-dev \
    pcre-dev && \
    : && \
    printf "\n" | pecl install imagick memcached mongodb oauth psr redis swoole xdebug && \
    docker-php-ext-enable imagick memcached mongodb oauth psr redis swoole xdebug && \
    : && \
    rm -rf /tmp/* /var/cache/apk/*

# PDFlib
# https://www.pdflib.com/download/pdflib-product-family/
RUN set -ex && \
    cd /tmp && \
    export PHP_EXT_DIR=$(php-config --extension-dir) && \
    curl -L -o pdflib.tar.gz https://www.pdflib.com/binaries/PDFlib/930/PDFlib-9.3.0-Alpine-Linux-x86_64-php.tar.gz && \
    tar -xvf pdflib.tar.gz && \
    mv PDFlib-* pdflib && cd pdflib && \
    cp bind/php/php-740-nts/php_pdflib.so $PHP_EXT_DIR && \
    docker-php-ext-enable php_pdflib && \
    cd /tmp && \
    rm -rf /tmp/*
