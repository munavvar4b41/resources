# Mailpit

Local mail catcher. SMTP mail is stored here and shown in a web inbox. The Laravel and WordPress guides send mail to this service.

## How to use

1. Copy the compose file below to the project root as `docker-compose.yml`.
2. Replace every `{project}` with a short name, such as `acme`. Container names cannot contain `{` or `}`.
3. Run `docker compose up -d`.
4. Open http://localhost:8025

Applications on the host use SMTP host `127.0.0.1` and port `1025`. Containers in another compose project use host `host.docker.internal` and port `1025`, with this host mapping on the app service:

```yaml
extra_hosts:
  - "host.docker.internal:host-gateway"
```

## URLs

- Web inbox: http://localhost:8025
- SMTP: `127.0.0.1:1025`

## Docker compose file

```yaml
services:
  mailpit:
    image: docker.io/axllent/mailpit:latest
    container_name: "{project}-mailpit"
    restart: unless-stopped
    ports:
      - "1025:1025"
      - "8025:8025"
```
