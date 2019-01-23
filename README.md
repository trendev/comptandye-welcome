# welcome
Welcome Page of comptandye, based on docker images (wordpress/apache/php + mysql)

## 1. Pre-requisit : DB initialization
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

#### ** Control the "home" / "siteurl" values ** :warning:
It should be "http://localhost:8000" running on DEV (local) environment and "https://www.comptandye.fr" running on PRODUCTION environment

`` echo 'select option_name,option_value  from wp_options where option_name="home" or option_name="siteurl";' | docker exec -i db-wp sh -c 'exec mysql -hdb-wp -P3306 -uroot -pnfY7.hXRcs --default-character-set=utf8 wordpress' ``

#### Change "home" / "siteurl" values (_if required_)
##### **PROD usage**

`` echo 'update wp_options set option_value="https://www.comptandye.fr" where option_name="home" or option_name="siteurl";' | docker exec -i db-wp sh -c 'exec mysql -hdb-wp -P3306 -uroot -pnfY7.hXRcs --default-character-set=utf8 wordpress' ``

You should also control the Apache load balancing settings...

##### *DEV/Test usage*

`` echo 'update wp_options set option_value="http://localhost:8000" where option_name="home" or option_name="siteurl";' | docker exec -i db-wp sh -c 'exec mysql -hdb-wp -P3306 -uroot -pnfY7.hXRcs --default-character-set=utf8 wordpress' ``

### Stop the MySQL container and go back to the main folder
`docker-compose down`

`cd ..`

*MySQL server is now stopped and the DB is prepared/ready for WordPress*

## 2. Build the WordPress platform

### 2.1 Control the `FORCE_SSL_ADMIN` definition :warning:
_Comment/Uncomment_ the line, located near line 89, ` define('FORCE_SSL_ADMIN', true);` in `wp-config.php` file _adding/removing_ a hashtag (`'#'`) at the begin of the line.

If the line is uncommented, the WordPress Admin Dashboard will be forced to use secured `https` protocol instead of unsecured `http` protocol, blocking access to `wp-admin` from unsecured request (users accessing `wp-admin` from localhost for exemple).

### 2.2 Build/Start the services 

`docker-compose up -d`

## 3. Backup the WordPress DB
### 3.1. Stop the WordPress running instance
`docker-compose stop wordpress`

### 3.2. Backup the running setting
`` docker run -it --link db-wp:mysql --network welcome_default --rm mysql:5.7 sh -c 'exec mysqldump -hdb-wp -P3306 -uroot -pnfY7.hXRcs wordpress' | tail -n +2 > db-backup.sql ``

### 3.3. Restart the WordPress instance
`docker-compose start wordpress`

## Admin the Production DB
`` docker run -it --link db-wp:mysql --network welcome_default --rm mysql:5.7 sh -c 'exec mysql -hdb-wp -P3306 -uroot -pnfY7.hXRcs --default-character-set=utf8 wordpress' ``
