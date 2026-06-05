# DEV Infra

This is a docker compose file for a local development environment. It include mailpit.

## How to use

1. Copy the below docker compose code to your project directory and name it `docker-compose.yml`
2. Run `docker compose up -d`
3. Access the applications at the following URLs:
   - Mailpit: http://localhost:8025

## Docker compose file

```docker
services:
  mailpit:
    image: axllent/mailpit:latest
    container_name: mailpit
    restart: unless-stopped
    ports:
      - "1025:1025"   # SMTP
      - "8025:8025"   # Web UI
```
