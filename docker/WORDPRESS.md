# WordPress

Local WordPress stack with Apache, MariaDB, and phpMyAdmin.

## How to use

1. Create `wordpress` and `docker/storage`. WordPress core is stored in `wordpress/`. MariaDB can read dumps placed in `docker/storage` at `/mnt/storage`.
2. Copy the compose file below to the project root as `docker-compose.yml`.
3. Replace every `{project}` with a short name, such as `acme`. Container names cannot contain `{` or `}`.
4. Run `docker compose up -d`.
5. Open the site and finish the installer.

`wordpress/` must be empty on the first start. The image copies WordPress core into that directory only when it has no core files yet.

## URLs

- WordPress: http://localhost:8000
- phpMyAdmin: http://localhost:8080
- MariaDB from the host: `127.0.0.1:3306`

Database name `wordpress`, user `wordpress`, password `wordpress`. The MariaDB root password is `root`.

The site directory and the storage directory each use `:Z` because only one container mounts them.

To send mail through the Mailpit compose file in `DEV_INFRA.md`, point an SMTP plugin at `host.docker.internal` on port `1025`. The WordPress service already maps that hostname to the host.

## Docker compose file

```yaml
services:
  wordpress:
    image: docker.io/library/wordpress:php8.5-apache
    container_name: "{project}-wordpress"
    restart: unless-stopped
    ports:
      - "8000:80"
    environment:
      WORDPRESS_DB_HOST: mariadb
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - ./wordpress:/var/www/html:Z
    extra_hosts:
      - "host.docker.internal:host-gateway"
    depends_on:
      mariadb:
        condition: service_healthy

  mariadb:
    image: docker.io/library/mariadb:latest
    container_name: "{project}-mariadb"
    restart: unless-stopped
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    volumes:
      - mariadb_data:/var/lib/mysql
      - ./docker/storage:/mnt/storage:Z
    healthcheck:
      test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]
      interval: 5s
      timeout: 5s
      retries: 20
      start_period: 20s

  phpmyadmin:
    image: docker.io/library/phpmyadmin:latest
    container_name: "{project}-phpmyadmin"
    restart: unless-stopped
    ports:
      - "8080:80"
    environment:
      PMA_HOST: mariadb
      PMA_PORT: 3306
    depends_on:
      mariadb:
        condition: service_healthy

volumes:
  mariadb_data:
```
