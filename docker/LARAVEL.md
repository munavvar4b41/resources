# Laravel

Local Laravel stack with PHP-FPM, Nginx, MariaDB, Vite, a queue listener, and phpMyAdmin.

## How to use

1. From the Laravel project root, create `docker/storage` for database dumps and other files you want available inside MariaDB at `/mnt/storage`.
2. Save the compose file as `docker-compose.yml`.
3. Save the Nginx config as `docker/nginx.conf`.
4. Save the image definition as `docker/Dockerfile`.
5. Replace every `{project}` with a short name, such as `acme`. Image and container names cannot contain `{` or `}`.
6. Copy `.env.example` to `.env` and set the values in the environment section below.
7. In `vite.config.js`, set the dev server so the browser on the host can reach Vite.
8. Install dependencies, then start the stack:

```bash
docker compose build
docker compose run --rm app composer install
docker compose run --rm app npm install
docker compose run --rm app php artisan key:generate
docker compose up -d
docker compose exec app php artisan migrate
```

## URLs

- Application: http://localhost:8000
- Vite: http://localhost:5173
- phpMyAdmin: http://localhost:8081 (user `root`, password `root`)

MariaDB is only on the compose network, at host `mariadb` and port `3306`.

The project directory is mounted with `:z` because app, Nginx, Vite, and the queue share it. Reverb uses that same mount when you add it. A mount used by one container, such as `docker/nginx.conf` and `docker/storage`, uses `:Z`.

## Environment

```dotenv
APP_URL=http://localhost:8000

DB_CONNECTION=mariadb
DB_HOST=mariadb
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=root
```

To capture mail with the Mailpit compose file in `DEV_INFRA.md`, add this to the `app` and `queue` services:

```yaml
extra_hosts:
  - "host.docker.internal:host-gateway"
```

```dotenv
MAIL_MAILER=smtp
MAIL_HOST=host.docker.internal
MAIL_PORT=1025
MAIL_SCHEME=null
```

## Vite

```js
server: {
    host: "0.0.0.0",
    port: 5173,
    hmr: {
        host: "localhost",
    },
},
```

## Docker compose file

```yaml
services:
  app:
    build:
      context: .
      dockerfile: docker/Dockerfile
    image: "{project}-php"
    container_name: "{project}-app"
    restart: unless-stopped
    working_dir: /var/www/html
    volumes:
      - ./:/var/www/html:z
    depends_on:
      mariadb:
        condition: service_healthy

  nginx:
    image: docker.io/library/nginx:latest
    container_name: "{project}-nginx"
    restart: unless-stopped
    ports:
      - "8000:80"
    volumes:
      - ./:/var/www/html:z
      - ./docker/nginx.conf:/etc/nginx/conf.d/default.conf:Z
    depends_on:
      - app

  mariadb:
    image: docker.io/library/mariadb:latest
    container_name: "{project}-mariadb"
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: root
    volumes:
      - mariadb_data:/var/lib/mysql
      - ./docker/storage:/mnt/storage:Z
    healthcheck:
      test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]
      interval: 5s
      timeout: 5s
      retries: 20
      start_period: 20s

  vite:
    build:
      context: .
      dockerfile: docker/Dockerfile
    image: "{project}-php"
    container_name: "{project}-vite"
    restart: unless-stopped
    working_dir: /var/www/html
    command: npm run dev -- --host 0.0.0.0 --port 5173
    volumes:
      - ./:/var/www/html:z
    ports:
      - "5173:5173"

  queue:
    build:
      context: .
      dockerfile: docker/Dockerfile
    image: "{project}-php"
    container_name: "{project}-queue"
    restart: unless-stopped
    working_dir: /var/www/html
    command: php artisan queue:listen --tries=1
    volumes:
      - ./:/var/www/html:z
    depends_on:
      mariadb:
        condition: service_healthy
      app:
        condition: service_started

  phpmyadmin:
    image: docker.io/library/phpmyadmin:latest
    container_name: "{project}-phpmyadmin"
    restart: unless-stopped
    ports:
      - "8081:80"
    environment:
      PMA_HOST: mariadb
      PMA_PORT: 3306
    depends_on:
      mariadb:
        condition: service_healthy

volumes:
  mariadb_data:
```

`queue:listen` reloads application code while the container keeps running. The PHP services share one image, so a later build uses the Docker cache.

## Reverb

Add this service when the application broadcasts with Laravel Reverb. The process listens on port 8080, and that port is published on the host.

```yaml
  reverb:
    build:
      context: .
      dockerfile: docker/Dockerfile
    image: "{project}-php"
    container_name: "{project}-reverb"
    restart: unless-stopped
    working_dir: /var/www/html
    command: php artisan reverb:start --host=0.0.0.0 --port=8080
    volumes:
      - ./:/var/www/html:z
    ports:
      - "8080:8080"
    depends_on:
      mariadb:
        condition: service_healthy
      app:
        condition: service_started
```

Install Reverb from the app container:

```bash
docker compose exec app php artisan install:broadcasting --reverb
```

That command writes `REVERB_APP_ID`, `REVERB_APP_KEY`, and `REVERB_APP_SECRET`, and it sets `REVERB_HOST="localhost"` with `REVERB_PORT=8080`. Set `REVERB_HOST` to `reverb` so the application and the queue publish to that service. Leave `VITE_REVERB_HOST` as `localhost` so the browser uses the published port.

```dotenv
BROADCAST_CONNECTION=reverb

REVERB_HOST=reverb
REVERB_PORT=8080
REVERB_SCHEME=http

VITE_REVERB_APP_KEY="${REVERB_APP_KEY}"
VITE_REVERB_HOST=localhost
VITE_REVERB_PORT="${REVERB_PORT}"
VITE_REVERB_SCHEME="${REVERB_SCHEME}"
```

Restart the Vite container after changing the `VITE_REVERB_*` values. The browser connects to ws://localhost:8080.

## docker/nginx.conf

```nginx
server {
    listen 80;
    root /var/www/html/public;
    index index.php;
    server_name localhost;
    client_max_body_size 64M;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass app:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```

## docker/Dockerfile

```dockerfile
FROM php:8.5-fpm

RUN apt-get update && apt-get install -y \
    $PHPIZE_DEPS \
    git \
    unzip \
    zip \
    curl \
    libzip-dev \
    libpng-dev \
    libjpeg62-turbo-dev \
    libfreetype6-dev \
    libwebp-dev \
    libicu-dev \
    libonig-dev \
    libxml2-dev \
    libpq-dev \
    supervisor \
    cron \
    nano \
    && rm -rf /var/lib/apt/lists/*

RUN docker-php-ext-configure gd \
    --with-freetype \
    --with-jpeg \
    --with-webp

RUN docker-php-ext-install \
    pdo_mysql \
    pdo_pgsql \
    zip \
    bcmath \
    intl \
    gd \
    pcntl \
    exif

RUN pecl install redis \
    && docker-php-ext-enable redis

RUN curl -fsSL https://deb.nodesource.com/setup_24.x | bash - \
    && apt-get install -y nodejs \
    && rm -rf /var/lib/apt/lists/*

COPY --from=docker.io/library/composer:latest /usr/bin/composer /usr/bin/composer

WORKDIR /var/www/html

CMD ["php-fpm"]
```
