# Podman SELinux labels

On Podman, and on Docker when SELinux is enforcing (Fedora, RHEL, Rocky Linux, AlmaLinux), a bind mount needs a label suffix. `:Z` and `:z` tell the engine how to label the host path.

## Comparison

| Option | Label | Use it when |
| ------ | ----- | ----------- |
| `:Z` | Private | One container mounts the path |
| `:z` | Shared | Several containers mount the same path |

## `:Z`

```yaml
volumes:
  - ./wordpress:/var/www/html:Z
```

The path is labeled for that one container. The HTML site root, the WordPress directory, `docker/nginx.conf`, `docker/storage`, and `searxng/settings.yml` each use `:Z` for this reason.

A second container that mounts the same path can be denied by SELinux:

```text
Container A → works
Container B → permission denied
```

## `:z`

```yaml
volumes:
  - ./:/var/www/html:z
```

The path is labeled so every container may use it, subject to normal file permissions. The Laravel project directory uses `:z` because app, Nginx, Vite, and the queue all mount it.

```text
project/
├── app
├── nginx
├── vite
└── queue
```

```yaml
app:
  volumes:
    - ./:/var/www/html:z

nginx:
  volumes:
    - ./:/var/www/html:z

vite:
  volumes:
    - ./:/var/www/html:z

queue:
  volumes:
    - ./:/var/www/html:z
```

A file inside that tree which only one container mounts, such as `docker/nginx.conf`, still uses `:Z`.
