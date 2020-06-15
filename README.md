# Magento 2 CloudDocker for Open Source Edition

Inspired by [magento/magento-cloud-docker](https://github.com/magento/magento-cloud-docker)

I extracted official Dockerfile files, applied some minor fixes and built a docker-composer.yml capable of running open source edition 

Configured with persistent storage

[Quickstart Guide](QUICKSTART.md)

[New Relic Guide](NEWRELIC.md)

## Usage

Extract Magento open source to `/htdocs`

### Generic

Pull

    docker-compose pull

Up / Down / Start / Stop

    docker-compose up -d
    docker-compose down -v
    docker-compose start
    docker-compose stop
    
Restart

    docker-compose restart
    
### Bash

    docker-compose run --rm cli
    
Or

    docker-compose run --rm cli bash 
    
### Composer

#### Install command

    docker-compose run --rm cli composer-installer vendor/module

#### Indirectly

    docker-compose run --rm cli 
    
Defaults inside `/app`
   
    composer --version
    composer install -vv
    composer require vendor/module:version -vv
    
#### Directly

    docker-compose run --rm cli composer --version
    docker-compose run --rm cli composer install -vv
    docker-compose run --rm cli composer require vendor/module:version -vv

Warning: permissions

### Magento command

    docker-compose run --rm cli magento-command

### Run cron

#### Container binary 

    docker-compose run --rm cli run-cron

#### Or safer - command within container

    docker-compose run --rm cli magento-command cron:run
    
#### Or even safer - configure cron container

```yml
  cron:
    hostname: cron.magento2.docker
    image: 'domw/magento2-cloud-php:7.2-cli'
    extends: generic
    command: run-cron
    environment:
      CRONTAB: '* * * * * root cd /app && /usr/local/bin/php bin/magento cron:run >> /app/var/log/cron.log'
    volumes:
      - 'app:/app'
    networks:
      magento:
        aliases:
          - cron.magento2.docker   
```

### Run CLI installer

With files extracted to `/htdocs` run CLI install process

    docker-compose run --rm cli magento-command setup:install --admin-firstname Admin --admin-lastname User [...]

### Flush redis

    docker-compose exec redis redis-cli FLUSHALL
    
### Flush varnish

    docker-compose exec varnish varnishadm ban req.url '~' '.'
    
### Access DB CLI

    docker-compose exec db sh -c 'mysql -u magento2 -pmagento2 magento2 "$@"'

### Optional 

Configure magento to use redis, elasticsearch and varnish if you choose to run these containers

## Build

### nginx

- [Docker Hub](https://hub.docker.com/r/domw/magento2-cloud-nginx)

- [Git](https://github.com/DominicWatts/Magento2CloudNginx)

nginx versions: 1.9, 1.10

### php-fpm

- [Docker Hub](https://hub.docker.com/r/domw/magento2-cloud-php)

- [Git](https://github.com/DominicWatts/Magento2CloudPHP)

php versions: 7.1, 7.2, 7.3, 7.4

### php-cli

- [Docker Hub](https://hub.docker.com/r/domw/magento2-cloud-php)

- [Git](https://github.com/DominicWatts/Magento2CloudPHP)

php versions: 7.1, 7.2, 7.3, 7.4

### tls

- [Docker Hub](https://hub.docker.com/r/domw/magento2-cloud-tls)

- [Git](https://github.com/DominicWatts/Magento2CloudTLS)

### varnish

- [Docker Hub](https://hub.docker.com/r/domw/magento2-cloud-varnish)

- [Git](https://github.com/DominicWatts/Magento2CloudVarnish)

varnish versions: 4.0, 6.2

### elasticsearch

- [Docker Hub](https://hub.docker.com/r/domw/magento2-cloud-elasticsearch)

- [Git](https://github.com/DominicWatts/Magento2CloudElasticsearch)

elasticsearch versions: 1.7, 2.4, 5.2, 6.5, 6.8, 7.5, 7.6

## Useful Resources
- [Docker Hub](https://hub.docker.com/r/domw/)

## Example stack

- [Docker Composer](https://github.com/DominicWatts/Magento2CloudDocker/blob/master/docker-compose.yml)

```
version: '2'
services:
  db:
    image: 'mariadb:10.4'
    restart: 'always'
    environment:
      - MYSQL_ROOT_PASSWORD=magento2
      - MYSQL_DATABASE=magento2
      - MYSQL_USER=magento2
      - MYSQL_PASSWORD=magento2
    hostname: db.magento2.docker
    ports:
      - '3306'
    networks:
      magento:
        aliases:
          - db.magento2.docker
    volumes:
      - ./mysql/data:/var/lib/mysql
      - ./mysql/conf:/etc/mysql/conf.d
  # redis:
  #   image: 'redis:3.0'
  #   volumes:
  #     - /data
  #   ports:
  #     - 6379
  #   networks:
  #     - magento
  # elasticsearch:
  #   image: 'domw/magento2-cloud-elasticsearch:7.6'
  #   networks:
  #     - magento      
  # phpmyadmin:
  #   image: phpmyadmin/phpmyadmin
  #   restart: 'always'
  #   ports:
  #     - 8000:80
  #   networks:
  #     magento:
  #       aliases:
  #         - phpmyadmin.magento2.docker    
  #   links: 
  #     - db:db
  #   environment:
  #     - MYSQL_USER=magento2
  #     - MYSQL_PASSWORD=magento2
  #     - MYSQL_ROOT_PASSWORD=magento2   
  web:
    image: 'domw/magento2-cloud-nginx:1.9'
    extends: generic
    hostname: web.magento2.docker
    restart: 'always'
    depends_on:
      - fpm
    volumes:
      - 'app:/app'
    networks:
      magento:
        aliases:
          - web.magento2.docker
  fpm:
    image: 'domw/magento2-cloud-php:7.2-fpm'
    extends: generic
    restart: 'always'
    ports:
      - 9000
    depends_on:
      - db
    volumes:
      - 'app:/app'
    networks:
      - magento
  cli:
    image: 'domw/magento2-cloud-php:7.2-cli'
    extends: generic
    hostname: deploy.magento2.docker
    depends_on:
      - db
    volumes:
      - 'app:/app'
      - '~/.composer/cache:/root/.composer/cache:delegated'
    networks:
      magento:
        aliases:
          - deploy.magento2.docker      
  varnish:
    image: 'domw/magento2-cloud-varnish:6.2'
    restart: 'always'
    environment:
      - VIRTUAL_HOST=magento2.docker
      - VIRTUAL_PORT=80
      - HTTPS_METHOD=noredirect
    ports:
      - '80:80'
    depends_on:
      - web
    networks:
      magento:
        aliases:
          - magento2.docker
  tls:
    image: 'domw/magento2-cloud-tls:latest'
    restart: 'always'
    ports:
      - '443:443'
    external_links:
      - 'varnish:varnish'
    depends_on:
      - varnish
    networks:
      - magento    
  # cron:
  #   hostname: cron.magento2.docker
  #   image: 'domw/magento2-cloud-php:7.2-cli'
  #   extends: generic
  #   command: run-cron
  #   environment:
  #     CRONTAB: '* * * * * root cd /app && /usr/local/bin/php bin/magento cron:run >> /app/var/log/cron.log'
  #   volumes:
  #     - 'app:/app'
  #   networks:
  #     magento:
  #       aliases:
  #         - cron.magento2.docker
  # mail:
  #   image: mailhog/mailhog
  #   restart: 'always'
  #   ports:
  #     - 1025:1025
  #     - 8025:8025
  #   links:
  #     - fpm
  #     - db
  #   networks:
  #     - magento
  generic:
    image: alpine
    environment:
      - PHP_MEMORY_LIMIT=2048M
      - UPLOAD_MAX_FILESIZE=64M
      - MAGENTO_ROOT=/app
      - PHP_IDE_CONFIG=serverName=magento_cloud_docker
      - XDEBUG_CONFIG=remote_host=host.docker.internal
      - MAGENTO_CLOUD_RELATIONSHIPS=eyJkYXRhYmFzZSI6W3siaG9zdCI6ImRiIiwicGF0aCI6Im1hZ2VudG8yIiwicGFzc3dvcmQiOiJtYWdlbnRvMiIsInVzZXJuYW1lIjoibWFnZW50bzIiLCJwb3J0IjoiMzMwNiJ9XSwicmVkaXMiOlt7Imhvc3QiOiJyZWRpcyIsInBvcnQiOiI2Mzc5In1dLCJlbGFzdGljc2VhcmNoIjpbeyJob3N0IjoiZWxhc3RpY3NlYXJjaCIsInBvcnQiOiI5MjAwIn1dfQ==
      - MAGENTO_CLOUD_ROUTES=eyJodHRwOlwvXC9tYWdlbnRvMi5kb2NrZXJcLyI6eyJ0eXBlIjoidXBzdHJlYW0iLCJvcmlnaW5hbF91cmwiOiJodHRwOlwvXC97ZGVmYXVsdH0ifSwiaHR0cHM6XC9cL21hZ2VudG8yLmRvY2tlclwvIjp7InR5cGUiOiJ1cHN0cmVhbSIsIm9yaWdpbmFsX3VybCI6Imh0dHBzOlwvXC97ZGVmYXVsdH0ifX0=
      - MAGENTO_CLOUD_VARIABLES=eyJBRE1JTl9FTUFJTCI6ImFkbWluQGV4YW1wbGUuY29tIiwiQURNSU5fUEFTU1dPUkQiOiIxMjMxMjNxIiwiQURNSU5fVVJMIjoiYWRtaW4ifQ==
      - MAGENTO_RUN_MODE=default
      - 'PHP_EXTENSIONS=bcmath bz2 calendar exif gd gettext intl mysqli pcntl pdo_mysql soap sockets sysvmsg sysvsem sysvshm opcache zip redis xsl ioncube'
volumes:
  app:
    driver_opts:
      type: none
      device: '${PWD}/htdocs'
      o: bind
networks:
  magento:
    driver: bridge
```

## Notes

Optional components

  - cron
  - phpmyadmin
  - elasticsearch
  - redis
  - mailhog
  
### Tweaks

PHP settings can be adjusted via

    `./htdocs/php.ini`
  
## Mailhog

### Container

```yml
  mail:
    image: mailhog/mailhog
    restart: 'always'
    ports:
      - 1025:1025
      - 8025:8025
    links:
      - fpm
      - db
    networks:
      - magento
```

### SMTP extension config

  -  host: `mail`
  -  port: `1025`
  -  protocol: `none`
  -  authentication: `plain`
  -  username/password: `[blank]`
