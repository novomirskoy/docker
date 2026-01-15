# Build extension installer
FROM alpine:latest AS php-extension-installer-builder
RUN apk add --no-cache wget
RUN wget -O /usr/bin/install-php-extensions https://github.com/mlocati/docker-php-extension-installer/releases/download/2.7.23/install-php-extensions \
    && chmod 755 /usr/bin/install-php-extensions

FROM php:8.4-cli

RUN apt update && apt install -y unzip bash locales \
    && apt-get clean autoclean \
    && apt-get autoremove -y \
    && rm -rf /var/lib/{apt,dpkg,cache,log}

# Timezone
ENV TZ=Europe/Moscow
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Locale
RUN sed -i -e \
  's/# ru_RU.UTF-8 UTF-8/ru_RU.UTF-8 UTF-8/' /etc/locale.gen \
   && locale-gen

ENV LANG=ru_RU.UTF-8
ENV LANGUAGE=ru_RU:ru
ENV LC_LANG=ru_RU.UTF-8
ENV LC_ALL=ru_RU.UTF-8

# Wait for it
COPY ./common/wait-for-it.sh /usr/local/bin/wait-for-it
RUN chmod 555 /usr/local/bin/wait-for-it

# Composer
COPY --from=composer/composer:2.8-bin /composer /usr/local/bin/composer

# Extension installer
COPY --from=php-extension-installer-builder /usr/bin/install-php-extensions /usr/local/bin/

# Install required extensions
RUN install-php-extensions opcache \
    intl \
    bcmath \
    gmp \
    xdebug \
    zip \
    bz2

# PHP settings
RUN mv $PHP_INI_DIR/php.ini-development $PHP_INI_DIR/php.ini
COPY ./common/conf.d $PHP_INI_DIR/conf.d
COPY ./8.4/conf.d $PHP_INI_DIR/conf.d

WORKDIR /src
