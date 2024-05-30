FROM php:8.3-apache

########################################################################################################################
# Copy VHost-Configuration into Container and enable ModRewrite
########################################################################################################################
COPY docker-config/000-default.conf /etc/apache2/sites-available/000-default.conf
COPY docker-config/php-extra.ini /usr/local/etc/php/conf.d/php-extra.ini
RUN mv /usr/local/etc/php/php.ini-development /usr/local/etc/php/php.ini \
    && a2enmod rewrite

########################################################################################################################
# Install tools: Git + Symfony Binary + Composer + Curl + Wget + Unzip
########################################################################################################################
RUN apt-get update \
    && apt-get install -y \
        curl \
        git \
        wget \
        unzip \
    && if [ -n "$HTTP_PROXY" ]; then git config --global http.proxy $HTTP_PROXY; fi \
    && rm -rf /var/lib/apt/lists/* \
    && curl -sS https://get.symfony.com/cli/installer | bash \
    && mv /root/.symfony5/bin/symfony /usr/local/bin/symfony \
    && curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin/ --filename=composer

########################################################################################################################
# Install PHP extensions via php-extension-installer (https://github.com/mlocati/docker-php-extension-installer)
########################################################################################################################
COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/bin/
RUN if [ -n "$HTTP_PROXY" ]; then pear config-set http_proxy $HTTP_PROXY; fi \
    && install-php-extensions gd xdebug ldap pdo_mysql opcache soap intl xsl zip exif

#######################################################################################################################
# Configure timezone and locale
#######################################################################################################################
ENV TZ="Europe/Berlin"
RUN apt-get update && apt-get install -y tzdata \
    && ln -fs /usr/share/zoneinfo/Europe/Berlin /etc/localtime \
    && dpkg-reconfigure --frontend=noninteractive tzdata
ENV LANG="de_DE.UTF-8"
ENV LANGUAGE="de_DE.UTF-8"
ENV LC_ALL="de_DE.UTF-8"
RUN apt-get update && apt-get install -y locales \
    && sed -i -e 's/# de_DE.UTF-8 UTF-8/de_DE.UTF-8 UTF-8/' /etc/locale.gen \
    && echo 'LANG="de_DE.UTF-8"'>/etc/default/locale \
    && dpkg-reconfigure --frontend=noninteractive locales \
    && update-locale LANG=de_DE.UTF-8
### date ### | ### printenv ### | ### locale -k LC_TIME