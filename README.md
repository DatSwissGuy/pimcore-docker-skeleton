# Pimcore Docker Compose Setup
This is a Docker Compose quick setup for a Pimcore skeleton installation.

# Step-by-Step

## Prerequisites
The starting point is a Docker-Compose configuration file (docker-compose.yml). The following is an example with some selected images: 

`Redis` => as a cache.

`MySQL` => as the database of choice. Feel free to replace it with MariaDB.

`Pimcore` => the official Pimcore php 7.4 Apache image.

`phpMyAdmin` => to make handling the database a bit easier without having to rely on tools like Sequel Pro, MySQL Workbench, etc.

`MailCatcher` => for E-Mail handling during development.

Change the ports as needed. I chose this setup to not interfere with other running containers.

```yml
version: "3.7"
services:
    cache:
        image: redis:latest
        volumes:
            - ./redis/config/redis.conf:/usr/local/etc/redis/redis.conf:ro
        entrypoint: redis-server /usr/local/etc/redis/redis.conf

    mysql:
        image: mysql
        volumes:
            - ./mysql/data/:/var/lib/mysql
        ports:
            - "7307:3306"
        environment:
            MYSQL_ROOT_PASSWORD: pimcore
            MYSQL_DATABASE: pimcore
            MYSQL_USER: pimcore
            MYSQL_PASSWORD: pimcore
        command: ['mysqld', '--character-set-server=utf8mb4', '--collation-server=utf8mb4_unicode_ci']

    pimcore:
        image: pimcore/pimcore:PHP7.4-apache
        volumes:
            - ./pimcore/config/php/php.ini:/usr/local/etc/php/php.ini         
            - ./pimcore/logs/apache/:/var/log/apache2
            - ./pimcore/www/:/var/www/html        
        ports:
            - "7080:80"
        # uncomment for ssl, beware this will need some extra configuration for certificates.
        #   - "8443:443"
    
    phpmyadmin:
        image: phpmyadmin/phpmyadmin
        ports:
            - "7081:80"
        environment:
          PMA_HOST: mysql
          PMA_USERNAME: root
          PMA_PASSWORD: pimcore


    mailcatcher:
        image: schickling/mailcatcher
        ports:
            - "1080:1080"
```

## Installation
The following is a sequence of commands and steps:

1. run `docker-compose up -d` 
2. run `docker ps` and identify the pimcore container
3. bash into container: `docker exec -it <pimcore-container> bash`
4. install package: `COMPOSER_MEMORY_LIMIT=-1 composer create-project pimcore/skeleton tmp`
5. move temporary installed packages to the correct folder in apache: `mv tmp/.[!.]* .` then `mv tmp/* .` then `rmdir tmp`
6. increase the memory_limit to >= 512MB as required by pimcore-install: `echo 'memory_limit = 512M' >> /usr/local/etc/php/conf.d/docker-php-memlimit.ini;`
7. restart / reload apache: `service apache2 reload`
8. on some machines there's gonna be issues with relative symlinked files, to solve this issue run: `chown www-data: . -R`
9. finally, time to install pimcore: `./vendor/bin/pimcore-install --mysql-host-socket=mysql --mysql-username=pimcore --mysql-password=pimcore --mysql-database=pimcore`
10. done!

The admin panel: http://localhost:7080/admin 
The frontend: http://localhost:7080
