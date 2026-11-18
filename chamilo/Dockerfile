
FROM php:8.3-apache

# 1. Install system dependencies
RUN apt-get update && apt-get install -y \
    libicu-dev libzip-dev libpng-dev \
    libjpeg62-turbo-dev libfreetype6-dev \
    libxml2-dev libonig-dev libldap2-dev \
    libcurl4-openssl-dev \
    libxapian-dev \
    build-essential pkg-config curl xz-utils \
    && rm -rf /var/lib/apt/lists/*

# 2. Configure GD
RUN docker-php-ext-configure gd \
    --with-freetype --with-jpeg

# 3. Install PHP extensions, including EXIF
RUN docker-php-ext-install -j$(nproc) \
    pdo_mysql \
    mysqli \
    intl \
    zip \
    gd \
    mbstring \
    bcmath \
    soap \
    ldap \
    exif

# 4. Install APCu
RUN pecl install apcu \
    && docker-php-ext-enable apcu

# 5. Build Xapian PHP bindings from source
WORKDIR /tmp

RUN curl -fSL \
    https://deb.debian.org/debian/pool/main/x/xapian-bindings/xapian-bindings_1.4.29.orig.tar.xz \
    -o xapian-bindings.tar.xz \
    && tar -xf xapian-bindings.tar.xz \
    && cd xapian-bindings-1.4.29 \
    && ./configure --with-php \
    && make -j$(nproc) \
    && make install \
    && docker-php-ext-enable xapian \
    && cd / \
    && rm -rf /tmp/xapian-bindings*

# 6. Configure Apache
RUN a2enmod rewrite headers expires

ENV APACHE_DOCUMENT_ROOT=/var/www/chamilo/public

RUN sed -ri 's!/var/www/html!/var/www/chamilo/public!g' \
    /etc/apache2/sites-available/*.conf \
    /etc/apache2/apache2.conf \
    /etc/apache2/conf-available/*.conf

# 7. PHP configuration
# 8. Verify required extensions
RUN php -m | grep -qi '^exif$' \
    && php -m | grep -qi '^xapian$'

# potential patch for redir bug:
COPY apache-chamilo.conf /etc/apache2/sites-available/000-default.conf

WORKDIR /var/www/chamilo

EXPOSE 80
