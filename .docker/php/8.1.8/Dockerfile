## Compile PHP
FROM alpine:latest

ARG USER_ID=1000
ARG PHP_VERSION=8.1.8

ENV PATH "/opt/php/bin:/opt/php/sbin:$PATH"

RUN apk add --no-cache shadow \
        tar \
        bzip2 \
        curl \
        curl-dev \
        autoconf \
        libjpeg-turbo-dev \
        libmcrypt-dev \
        c-client \
        libxml2-dev \
        openssl-dev \
        libpng-dev \
        freetype-dev \
        krb5-dev \
        libxslt-dev \
        gcc \
        icu-dev \
        icu-libs \
        icu \
        git \
        build-base \
        gettext-dev \
        ca-certificates \
        g++ \
        libsodium-dev \
        libgc++ \
        libzip-dev \
        zip \
        oniguruma-dev

RUN mkdir -p /opt/php/fpm /opt/php-src /var/www \
    && useradd -u ${USER_ID} --gid www-data --home-dir /var/www www-data -s /bin/sh -M \
    && chown www-data:www-data /var/www \
    && cd /opt/php-src \
    && curl https://www.php.net/distributions/php-${PHP_VERSION}.tar.bz2 -o php-${PHP_VERSION}.tar.bz2 \
    && tar -jxf php-${PHP_VERSION}.tar.bz2 \
    && cd /opt/php-src/php-${PHP_VERSION} \
    && ./configure \
      --prefix=/opt/php \
      --with-zlib-dir \
      --with-pear \
      --with-curl \
      --with-zlib \
      --with-mhash \
      --with-pdo-mysql \
      --with-mysqli \
      --with-openssl \
      --with-fpm-user=www-data \
      --with-fpm-group=www-data \
      --with-libdir=/usr/lib \
      --with-gettext \
      --with-xsl \
      --enable-mbstring \
      --enable-bcmath \
      --enable-fpm \
      --enable-intl \
      --enable-sockets \
      --disable-rpath \
      --without-sqlite3 \
      --without-pdo-sqlite \
      --without-pgsql \
      --with-sodium=shared \
    && make -j "$(nproc)"\
    && make install \
    && pecl install xdebug \
    && rm -Rf /opt/php-src \
    && rm -f /opt/php/bin/php-cgi \
    && rm -f /opt/php/bin/phpdbg \
    && rm -Rf /opt/php/lib/php/doc \
    && rm -Rf /opt/php/php/man

## Compile Nginx Module
FROM nginx:stable-alpine

RUN apk --update --no-cache add \
        gcc \
        make \
        libc-dev \
        g++ \
        openssl-dev \
        linux-headers \
        pcre-dev \
        zlib-dev \
        libtool \
        automake \
        autoconf \
        git

RUN cd /opt \
    && git clone --depth 1 -b master --single-branch https://github.com/openresty/headers-more-nginx-module.git \
    && cd /opt/headers-more-nginx-module \
    && git submodule update --init \
    && cd /opt \
    && wget -O - http://nginx.org/download/nginx-1.22.0.tar.gz | tar zxfv - \
    && mv /opt/nginx-1.22.0 /opt/nginx \
    && cd /opt/nginx \
    && ./configure --with-compat --add-dynamic-module=/opt/headers-more-nginx-module \
    && make modules \
    && rm -rf /tmp

## Run Nginx and PHP FPM
FROM nginx:stable-alpine

ARG USER_ID=1000
ARG INSTALL_COMPOSER=0

ENV PATH "/opt/php/bin:/opt/php/sbin:$PATH"
ENV PS1="\u@\w $ "

RUN apk add --no-cache shadow git curl supervisor icu-libs \
    && useradd -u ${USER_ID} -g www-data -d /var/www www-data -s /bin/sh -M \
    && rm -Rf /var/cache/apk \
    && echo "fs.inotify.max_user_watches=524288" > /etc/sysctl.d/99-sysctl.conf \
    && mkdir -p /var/www/html /var/log/nginx /var/log/php \
    && chown -R :www-data /var/log \
    && touch /var/log/php/www.access.log

COPY --from=0 ["/usr/lib/libexslt.so.0", "/usr/lib/libonig.so.5", "/usr/lib/libstdc++.so.6", "/usr/lib/libintl.so.8", "/usr/lib/libpng16.so.16", "/usr/lib/libjpeg.so.8", "/usr/lib/libcurl.so.4", "/usr/lib/libfreetype.so.6", "/usr/lib/libicuio.so.71", "/usr/lib/libicui18n.so.71", "/usr/lib/libicuuc.so.71", "/usr/lib/libgcc_s.so.1", "/usr/lib/libxml2.so.2", "/usr/lib/libxslt.so.1", "/usr/lib/libgcrypt.so.20", "/usr/lib/libnghttp2.so.14", "/usr/lib/libbz2.so.1", "/usr/lib/libicudata.so.71", "/usr/lib/libgpg-error.so.0", "/usr/lib/libzip.so.5", "/usr/lib/liblzma.so.5", "/usr/lib/"]
COPY --from=0 ["/opt/php", "/opt/php"]
COPY --from=1 ["/opt/nginx/objs/ngx_http_headers_more_filter_module.so", "/etc/nginx/modules"]
COPY [".docker/nginx/certs", "etc/nginx/certs"]
COPY [".docker/nginx/nginx.conf", "/etc/nginx/nginx.conf"]
COPY [".docker/nginx/fastcgi_params", "/etc/nginx/fastcgi_params"]
COPY [".docker/nginx/localhost.conf", "/etc/nginx/sites-enable/0-localhost.conf"]
COPY [".docker/php/8.1.8/fpm/php-fpm.conf", "/opt/php/etc/"]
COPY [".docker/php/8.1.8/fpm/php.ini", "/opt/php/lib/"]
COPY [".docker/php/8.1.8/fpm/www.conf", "/opt/php/etc/php-fpm.d/"]
COPY [".docker/supervisor/supervisord.conf", "/etc/supervisord.conf"]

RUN if [ "$INSTALL_COMPOSER" = "1" ]; then \
        php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
        && php composer-setup.php \
        && php -r "unlink('composer-setup.php');" \
        && mv composer.phar /usr/local/bin/composer; \
    fi \
    && mkdir -p /var/log/supervisord

WORKDIR /var/www/html

EXPOSE 80 443

CMD [ "/usr/bin/supervisord", "-c", "/etc/supervisord.conf" ]
