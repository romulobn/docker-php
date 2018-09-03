FROM php:7.2-cli

ARG TZ=UTC
ENV TZ ${TZ}
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Core dependencies
RUN apt-get update && \
    apt-get install -y apt-utils && \
    pecl channel-update pecl.php.net && \
    apt-get install -y --no-install-recommends \
    curl \
    libssh2-1-dev \
    libmemcached-dev \
    libicu-dev \
    libz-dev \
    libpq-dev \
    libjpeg-dev \
    libpng-dev \
    libfreetype6-dev \
    libssl-dev \
    libmcrypt-dev

# Install the PHP libssh2 extention
RUN pecl install ssh2-1.1.2 && \
    docker-php-ext-enable ssh2

# Install the PHP libssh2 extention
RUN docker-php-ext-install pdo_mysql

# Install the PHP gd library
RUN docker-php-ext-configure gd \
    --enable-gd-native-ttf \
    --with-jpeg-dir=/usr/lib \
    --with-freetype-dir=/usr/include/freetype2 && \
    docker-php-ext-install gd

# Install PHP mysqli extensions
RUN apt-get install -y mysql-client && \
    docker-php-ext-install mysqli

# Install PHP ZipArchive library
RUN apt-get install libzip-dev -y && \
    docker-php-ext-configure zip --with-libzip && \
    docker-php-ext-install zip

# Install PHP xDebug library
RUN pecl install xdebug && \
    docker-php-ext-enable xdebug

COPY ./xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini

RUN sed -i "s/xdebug.remote_autostart=0/xdebug.remote_autostart=1/" /usr/local/etc/php/conf.d/xdebug.ini && \
    sed -i "s/xdebug.remote_enable=0/xdebug.remote_enable=1/" /usr/local/etc/php/conf.d/xdebug.ini && \
    sed -i "s/xdebug.cli_color=0/xdebug.cli_color=1/" /usr/local/etc/php/conf.d/xdebug.ini

# Install PHP Memcached library
RUN curl -L -o /tmp/memcached.tar.gz "https://github.com/php-memcached-dev/php-memcached/archive/php7.tar.gz"; \
    mkdir -p memcached \
    && tar -C memcached -zxvf /tmp/memcached.tar.gz --strip 1 \
    && ( \
    cd memcached \
    && phpize \
    && ./configure \
    && make -j$(nproc) \
    && make install \
    ) \
    && rm -r memcached \
    && rm /tmp/memcached.tar.gz \
    && docker-php-ext-enable memcached

COPY ./php.ini /usr/local/etc/php/php.ini
COPY ./codeigniter.ini /usr/local/etc/php/conf.d


RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog

RUN curl -L -o /tmp/composer-installer.php "https://getcomposer.org/installer" && \
    php /tmp/composer-installer.php --install-dir=/usr/local/bin --filename=composer

CMD ["tail", "-f", "/dev/null"]