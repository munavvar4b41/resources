# HTML

Nginx serves a static site from the project directory.

## How to use

1. Copy the compose file below to the project root as `docker-compose.yml`.
2. Replace every `{project}` with a short name, such as `acme`. Container names cannot contain `{` or `}`.
3. Put the site files, including `index.html`, in the project root.
4. Run `docker compose up -d`.
5. Open http://localhost:8000

Nginx serves the project root. Keep secrets and credentials out of that directory.

The `:Z` suffix gives the mount a private SELinux label. Use it when only one container mounts the directory.

## Docker compose file

```yaml
services:
  html:
    image: docker.io/library/nginx:latest
    container_name: "{project}-html"
    restart: unless-stopped
    ports:
      - "8000:80"
    volumes:
      - ./:/usr/share/nginx/html:Z
```
