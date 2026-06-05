# Podman SELinux: Difference Between `:Z` and `:z`

When using Podman (or Docker on SELinux-enabled systems such as Fedora, RHEL, Rocky Linux, AlmaLinux, etc.), there is an important difference between `:Z` and `:z` when mounting host directories into containers.

## Quick Comparison

| Option | Meaning               | Use Case                                 |
| ------ | --------------------- | ---------------------------------------- |
| `:Z`   | Private SELinux label | One container exclusively uses the mount |
| `:z`   | Shared SELinux label  | Multiple containers share the mount      |

---

## `:Z` (Capital Z)

When you mount a volume using:

```yaml
volumes:
  - ./src:/var/www/html:Z
```

Podman relabels the directory with a **private SELinux context** intended for a single container.

### Example

```text
src/
└── Container A
```

If another container tries to mount the same directory, SELinux may deny access.

### Typical Symptoms

```text
Container A → Works
Container B → Permission denied
```

This can happen because the directory label is assigned specifically for the first container.

### When to Use `:Z`

Use `:Z` when a directory is mounted into only one container.

Example:

```yaml
backup:
  volumes:
    - ./backup:/backup:Z
```

---

## `:z` (Small z)

When you mount a volume using:

```yaml
volumes:
  - ./src:/var/www/html:z
```

Podman applies a **shared SELinux label** that allows multiple containers to access the same directory.

### Example

```text
src/
├── nginx
├── app
├── queue
├── reverb
└── node
```

All containers can read and write according to normal filesystem permissions.

### When to Use `:z`

Use `:z` whenever multiple containers need access to the same directory.

Example:

```yaml
app:
  volumes:
    - ./src:/var/www/html:z

nginx:
  volumes:
    - ./src:/var/www/html:z

queue:
  volumes:
    - ./src:/var/www/html:z

reverb:
  volumes:
    - ./src:/var/www/html:z
```

---
