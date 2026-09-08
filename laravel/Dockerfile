FROM php:8.4-apache-trixie

# 1. Install System Dependencies & Clear Cache
RUN apt-get update && apt-get install -qy --no-install-recommends \
    libzip-dev \
    zip \
    supervisor \
    cron \
    git \
    curl \
    libmcrypt-dev \
    ghostscript \
    libjpeg62-turbo-dev \
    libpng-dev \
    postgresql-client \
    libpq-dev \
    libjpeg-dev \
    libcurl4-openssl-dev \
    pkg-config \
    libssl-dev \
    libicu-dev \
    gettext \
    librabbitmq-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# 2. Configure Apache (DocumentRoot, mod_rewrite, port 8080)
ENV APACHE_DOCUMENT_ROOT=/var/www/html/public
RUN a2enmod rewrite \
    && sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf \
    && sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf \
    && sed -ri 's/^Listen 80$/Listen 8080/' /etc/apache2/ports.conf \
    && sed -ri 's/<VirtualHost \*:80>/<VirtualHost *:8080>/' /etc/apache2/sites-available/*.conf \
    && echo 'ServerName localhost' > /etc/apache2/conf-available/server-name.conf \
    && a2enconf server-name \
    && mkdir -p /var/run/apache2 /var/lock/apache2 \
    && chown -R www-data:www-data /var/run/apache2 /var/lock/apache2 /var/log/apache2

# 3. Install PHP Extensions
RUN docker-php-ext-configure gd --with-jpeg \
    && docker-php-ext-install pdo_mysql pdo_pgsql bcmath zip pcntl gd sockets intl \
    && pecl install raphf && docker-php-ext-enable raphf \
    && pecl install pecl_http && docker-php-ext-enable http \
    && pecl install amqp && docker-php-ext-enable amqp \
    && pecl install redis && docker-php-ext-enable redis

# 4. Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

WORKDIR /var/www/html