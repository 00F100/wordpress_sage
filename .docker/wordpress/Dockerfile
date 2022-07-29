FROM 00f100/php-nginx-wordpress

COPY [".", "/var/www/html"]
COPY [".docker/nginx/example.wordpress.org.conf", "/etc/nginx/sites-enable/1-example.wordpress.org.conf"]

CMD [ "/usr/bin/supervisord", "-c", "/etc/supervisord.conf" ]

WORKDIR /var/www/html