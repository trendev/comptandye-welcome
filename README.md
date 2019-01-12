# welcome
Welcome Page of comptandye, based on docker images (wordpress/apache/php + mysql)

## 1. Pre-requisit
### Build the MySQL container
`cd db/`

`docker-compose up -d`

*MySQL Server is now started*

### Restore the DB from the last WordPress backup
> **Should be skipped if you want to start from scratch**

> Assuming the last backup is in the file db-backup.sql

#### Connect as MySQL Root
`docker run -it --link db-wp:mysql --network db_default --rm mysql:5.7 sh -c 'exec mysql -hdb-wp -P3306 -uroot -pnfY7.hXRcs --default-character-set=utf8'`
##### Drop and Recreate the WordPress DB

> Enter the following 3 SQL commands

`drop database if exists wordpress;`

`CREATE DATABASE wordpress CHARACTER SET 'utf8';`

`quit;`

#### Restore the DB
`cat ../db-backup.sql | docker exec -i db-wp /usr/bin/mysql -u root --password=nfY7.hXRcs wordpress`

### Stop the MySQL container
`docker-compose down`

*MySQL server is now stopped and the DB is prepared/ready for WordPress*

## 2. Build the WordPress platform
`cd ..`

`docker-compose up -d`

## 3. Backup the WordPress DB
`docker-compose stop wordpress`

`` docker run -it --link db-wp:mysql --network welcome_default --rm mysql:5.7 sh -c 'exec mysqldump -hdb-wp -P3306 -uroot -pnfY7.hXRcs wordpress' | tail -n +2 > db-backup.sql ``

`docker-compose start wordpress`

## Log in MySQL server as root
`` docker run -it --link db-wp:mysql --network welcome_default --rm mysql:5.7 sh -c 'exec mysql -hdb-wp -P3306 -uroot -pnfY7.hXRcs --default-character-set=utf8 wordpress' ``
