# Wordpress + Sage

This project is example and uses [Wordpress](https://wordpress.org/) + [Sage](https://roots.io/sage/)

This project need [Docker](https://www.docker.com/) and [Docker Compose plugin](https://docs.docker.com/compose/install/compose-plugin/)

## Applications inside containers

| Application | Version |
|---|---|
| PHP | 8.1.8 |
| Composer | 2.3.10 |
| Node.js | v18.7.0 |
| Yarn | 1.22.19 |

## Development

### Install

For install in localhost environment (need sudo permission)

```
bin/install
```

### Execute

For execute containers with dabase and wordpress

```
bin/start
```

Now you have a [frontend](https://example.wordpress.org/) and [admin](https://example.wordpress.org/core/wp-admin) of Wordpress

> Default admin login is "root" for username and password

Stop execution containers

```
bin/stop
```

### More commands

Logs Containers

```
bin/logs
```

PHP Client with Composer

```
bin/php/cli
```

PHP Build

```
bin/php/build
```

PHP Logs

```
bin/php/logs
```

Node (Yarn) Client

```
bin/node/cli
```

Database Clean

```
bin/database/clean
```

## Production

### Security

All headers are configured for not expose server informations and block attacks.

Redirect HTTP to HTTPS
```
$ curl -X GET http://example.wordpress.org --head --insecure
HTTP/1.1 200 OK
Date: Fri, 29 Jul 2022 16:26:45 GMT
Content-Type: text/html; charset=UTF-8
Transfer-Encoding: chunked
Connection: keep-alive
Vary: Accept-Encoding
Content-Security-Policy: frame-ancestors 'self';
X-Content-Type-Options: nosniff
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Frame-Options: SAMEORIGIN
```

HTTPS
```
$ curl -X GET https://example.wordpress.org --head --insecure
HTTP/1.1 200 OK
Date: Fri, 29 Jul 2022 16:27:01 GMT
Content-Type: text/html; charset=UTF-8
Transfer-Encoding: chunked
Connection: keep-alive
Vary: Accept-Encoding
Vary: Accept-Encoding
Content-Security-Policy: frame-ancestors 'self';
X-Content-Type-Options: nosniff
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Frame-Options: SAMEORIGIN
```