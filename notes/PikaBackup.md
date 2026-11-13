# Pika Backup include and exclude

Open the backup under the **Backup** tab. Use **Files to Back Up** and **Exclude From Backup**. Changes apply to the next run. The same rules live in `~/.var/app/org.gnome.World.PikaBackup/config/pika-backup/backup.json` if you edit the file instead (close the app first).

## GUI — include

1. Open the backup configuration.
2. Under **Files to Back Up**, keep **Home** (shows as the home folder). That is `include: [""]`.
3. Optional: use **+** → **Include Folder** or **Include File** for paths outside home.

## GUI — exclude

1. Under **Exclude From Backup**, click **+**.
2. Apply the rules below.

### Suggested exclusions

1. Open **Suggested Exclusions**.
2. Enable **Caches** (XDG/Flatpak caches and `CACHEDIR.TAG` dirs).

### Unix filename patterns (Fnmatch)

1. Choose **Pattern**.
2. Set **Pattern Type** to **Unix Filename Pattern**.
3. Add each pattern, then confirm:

| Pattern | Why |
| ------- | --- |
| `*/storage/debugbar` | Laravel Debugbar cache under any project |
| `*/storage/framework` | Laravel framework cache, sessions, views |
| `*/storage/inertia-devtools` | Inertia DevTools storage |
| `*/public/storage` | Laravel storage |

`*` matches across `/`. Use **Edit Pattern** on an existing row to change it later. **Regular Expression** is a different type; do not use it for these three.

## Config JSON

```json
"include": [
  ""
],
"exclude": [
  {
    "Fnmatch": "*/storage/debugbar"
  },
  {
    "Fnmatch": "*/storage/framework"
  },
  {
    "Fnmatch": "*/storage/inertia-devtools"
  },
  {
    "Fnmatch": "*/public/storage"
  },
  {
    "PathPrefix": "/home/munavvar/.local/share/containers/storage"
  },
  "Caches"
]
```
