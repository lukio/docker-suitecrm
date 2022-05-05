FROM php:7.4.22-apache
MAINTAINER Luciano Rossi <lukio@gcoop.coop>

ENV DOWNLOAD_URL https://github.com/salesagility/SuiteCRM/archive/refs/tags/v7.10.36.zip
ENV DOWNLOAD_FILE v7.10.36.zip
ENV EXTRACT_FOLDER SuiteCRM-7.10.36
ENV WWW_FOLDER /var/www/html
ENV WWW_USER www-data
ENV WWW_GROUP www-data

RUN apt-get -yqq update && \
    apt-get -yqq install libcurl4-gnutls-dev libpng-dev libmcrypt-dev libssl-dev libc-client2007e-dev libkrb5-dev unzip cron re2c python tree wget libzip-dev telnet iproute2 build-essential libgmp-dev libicu-dev libedit-dev libreadline-dev libxml2-dev libonig-dev && \
    docker-php-ext-configure imap --with-imap-ssl --with-kerberos && \
    docker-php-ext-install bcmath gmp intl json readline opcache soap xml pdo_mysql mysqli curl gd zip mbstring imap && \
    rm -rf /var/lib/apt/lists/*

RUN apt-get clean

WORKDIR /tmp

# Instalo composer v1.10.5
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
RUN php -r "if (hash_file('sha384', 'composer-setup.php') === '$(wget -q -O - https://composer.github.io/installer.sig)') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
RUN php composer-setup.php --1
RUN php -r "unlink('composer-setup.php');"
RUN mv composer.phar /usr/local/bin/composer

RUN wget ${DOWNLOAD_URL}
RUN unzip $DOWNLOAD_FILE
RUN rm $DOWNLOAD_FILE
RUN rm -rf ${WWW_FOLDER}/*
RUN cp -R ${EXTRACT_FOLDER}/* ${WWW_FOLDER}/
RUN chown -R ${WWW_USER}:${WWW_GROUP} ${WWW_FOLDER}/*
RUN chown -R ${WWW_USER}:${WWW_GROUP} ${WWW_FOLDER}

COPY php.ini /usr/local/etc/php/php.ini
COPY config_override.php.pyt /usr/local/src/config_override.php.pyt
COPY envtemplate.py /usr/local/bin/envtemplate.py
COPY init.sh /usr/local/bin/init.sh
COPY config.php ${WWW_FOLDER}
RUN chown -R ${WWW_USER}:${WWW_GROUP} ${WWW_FOLDER}/config.php
RUN chmod 766 ${WWW_FOLDER}/config.php

RUN chmod u+x /usr/local/bin/init.sh
RUN chmod u+x /usr/local/bin/envtemplate.py

ADD crons.conf /root/crons.conf
RUN crontab /root/crons.conf

WORKDIR ${WWW_FOLDER}/
RUN composer install

WORKDIR /tmp

EXPOSE 80
ENTRYPOINT ["/usr/local/bin/init.sh"]
