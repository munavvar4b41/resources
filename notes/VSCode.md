# VS Code terminal → toolbox

Use a Fedora toolbox named `dev` as the integrated terminal. Same keys work in Cursor (`~/.config/Cursor/User/settings.json`) and VS Code (`~/.config/Code/User/settings.json`).

## GUI

1. Open the Command Palette (`Ctrl+Shift+P`) → **Preferences: Open Settings (UI)**.
2. Search for `terminal default profile linux`.
3. Under **Terminal › Integrated › Default Profile: Linux**, you need a `toolbox` profile first (settings UI cannot fully define custom args). Prefer the JSON steps below, or:
   - Command Palette → **Preferences: Open User Settings (JSON)**
   - Paste the config in the next section
   - Save, then open a new terminal (`Ctrl+Shift+\``) — it should run `toolbox enter dev`
4. To switch without changing the default: open the terminal dropdown → **Select Default Profile** → choose **toolbox** (after the profile exists).

Create the container once if it is missing:

```bash
toolbox create -c dev
```

## settings.json

```json
{
    "terminal.integrated.profiles.linux": {
        "toolbox": {
            "path": "/usr/bin/toolbox",
            "args": [
                "enter",
                "dev"
            ]
        }
    },
    "terminal.integrated.defaultProfile.linux": "toolbox"
}
```

| Setting | Role |
| ------- | ---- |
| `profiles.linux.toolbox` | Custom profile that runs `toolbox enter dev` |
| `defaultProfile.linux` | New terminals use that profile |

Change `"dev"` in `args` if the toolbox has another name. Existing terminals keep their old shell until you open a new one.
