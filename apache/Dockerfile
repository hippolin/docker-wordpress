FROM php:5.6-apache

RUN a2enmod rewrite

# install the PHP extensions we need
RUN apt-get update && apt-get install -y libpng12-dev libjpeg-dev && rm -rf /var/lib/apt/lists/* \
	&& docker-php-ext-configure gd --with-png-dir=/usr --with-jpeg-dir=/usr \
	&& docker-php-ext-install gd
RUN docker-php-ext-install mysqli

RUN curl -SL "http://xcache.lighttpd.net/pub/Releases/3.2.0/xcache-3.2.0.tar.gz" -o xcache.tar.gz \
    && mkdir -p /usr/src/xcache \
	&& tar -zxf xcache.tar.gz -C /usr/src/xcache --strip-components=1 \
	&& rm -f xcache.tar.gz \
	&& cd /usr/src/xcache \
	&& phpize \
	&& ./configure --enable-xcache \
	&& make install \
	&& cp xcache.ini /usr/local/etc/php/conf.d/docker-php-ext-xcache.ini \
	&& make clean

COPY cloudwatch_logs.conf /etc/apache2/conf-enabled/cloudwatch_logs.conf

VOLUME /var/www/html

ENV WORDPRESS_VERSION 4.3.1
ENV WORDPRESS_MD5 330b427b7c69b5fcd9943ed666ed8650

# upstream tarballs include ./wordpress/ so this gives us /usr/src/wordpress
RUN curl -o wordpress.tar.gz -SL https://tw.wordpress.org/wordpress-${WORDPRESS_VERSION}-zh_TW.tar.gz \
	&& echo "$WORDPRESS_MD5 *wordpress.tar.gz" | md5sum -c - \
	&& tar -xzf wordpress.tar.gz -C /usr/src/ \
	&& rm wordpress.tar.gz \
	&& chown -R www-data:www-data /usr/src/wordpress

COPY docker-entrypoint.sh /entrypoint.sh

# grr, ENTRYPOINT resets CMD now
ENTRYPOINT ["/entrypoint.sh"]
CMD ["apache2-foreground"]
