FROM php:7.2-fpm-alpine

ARG BUILD_DATE
ARG VCS_REF
ARG IMAGE_NAME
ARG DOCKER_REPO
ENV COMPOSER_ALLOW_SUPERUSER 1

LABEL Maintainer="Mhd Zaher Ghaibeh <z@zah.me>" \
      org.label-schema.name="zaherg/php-and-nginx:7.2" \
      org.label-schema.description="Lightweight container with Nginx 1.12 & PHP-FPM 7.2 based on Alpine Linux." \
      org.label-schema.url="https://www.zah.me" \
      org.label-schema.build-date=$BUILD_DATE \
      org.label-schema.vcs-url="https://github.com/linuxjuggler/php-and-nginx.git" \
      org.label-schema.vcs-ref=$VCS_REF \
      org.label-schema.schema-version="1.0.0"


COPY root/. /

RUN set -ex \
    echo "@community http://dl-4.alpinelinux.org/alpine/edge/community" >> /etc/apk/repositories && \
    echo "@testing http://dl-4.alpinelinux.org/alpine/edge/testing" >> /etc/apk/repositories && \
    apk update && apk upgrade && \
    apk add --no-cache git curl openssh-client ca-certificates icu rsyslog logrotate runit curl \
            icu libpng freetype libjpeg-turbo postgresql-dev libffi-dev && \
    apk add --no-cache --virtual build-dependencies icu-dev g++ make autoconf \
            libxml2-dev freetype-dev libpng-dev libjpeg-turbo-dev && \
    cd /tmp && \
    curl -Ls https://github.com/nimmis/docker-utils/archive/master.tar.gz | tar xfz - && \
    ./docker-utils-master/install.sh && \
    rm -Rf ./docker-utils-master && \
    docker-php-source extract && \
    pecl install redis opcache && \
    docker-php-ext-enable redis opcache && \
    docker-php-source delete && \
    docker-php-ext-configure pgsql -with-pgsql=/usr/local/pgsql && \
    docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ && \
    docker-php-ext-install -j$(nproc) intl sockets pgsql pdo_mysql pdo_pgsql zip gd bcmath && \
    sed  -i "s|\*.emerg|\#\*.emerg|" /etc/rsyslog.conf && \
    sed -i 's/$ModLoad imklog/#$ModLoad imklog/' /etc/rsyslog.conf && \
    sed -i 's/$KLogPermitNonKernelFacility on/#$KLogPermitNonKernelFacility on/' /etc/rsyslog.conf && \
    sed -i 's/user = www-data/user = nginx/' /usr/local/etc/php-fpm.d/www.conf && \
    sed -i 's/group = www-data/group = nginx/' /usr/local/etc/php-fpm.d/www.conf && \
    curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer && \
    apk add nginx && \
    mkdir /web && mkdir /run/nginx && \
    rm -rf /var/cache/apk/* && \
    apk del build-dependencies && \
    rm -rf /tmp/*

# Set environment variables.
ENV HOME /root

# Define working directory.
WORKDIR /root

COPY rootfs /

EXPOSE 80 443

# Define default command.
CMD ["/boot.sh"]
