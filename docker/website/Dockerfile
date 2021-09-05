FROM php:7.0-apache

RUN apt-get update \
	&& apt-get install -y \
	libzip-dev \
	zip \
	&& docker-php-ext-install zip pdo pdo_mysql

ENV APACHE_DOCUMENT_ROOT /opt/web/

RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf
RUN sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf

RUN mkdir -p ${APACHE_DOCUMENT_ROOT} && chown -R www-data.www-data ${APACHE_DOCUMENT_ROOT}
