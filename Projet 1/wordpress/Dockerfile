FROM wordpress:latest

RUN apt-get update \
    && apt-get install -y libpng-dev \
    && docker-php-ext-install gd

EXPOSE 80

CMD ["apache2-foreground"]
