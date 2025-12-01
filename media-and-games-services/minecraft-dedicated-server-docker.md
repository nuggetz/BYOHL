# 🚀 Minecraft Dedicated Server Installation Guide

## 🧱 1. Project Structure
Assume the following directory layout:

```txt
minecraft-server/
├── better-mc/               # One container per world/modpack → one sub‑directory
│   ├── docker-compose.yml
│   ├── .env
│   └── data/
```

## 🐳 2. docker‑compose.yml for CurseForge

```yml
version: '3.8'

services:
  mc:
    image: itzg/minecraft-server:latest
    container_name: minecraft-server-modpacks
    restart: unless-stopped
    tty: true
    stdin_open: true
    ports:
      - "25565:25565"   # default port – change it if you run another container on the same host
    env_file:
      - .env
    environment:
      EULA: "TRUE"
      TYPE: AUTO_CURSEFORGE
      CF_SLUG: better-mc-forge-bmc4
      MEMORY: "4G"
      EXCLUDE_CLIENT_MODS: "true"
      REMOVE_OLD_MODS: "true"
    volumes:
      - /path-to-data/data:/data
```

## 🔐 3. `.env` file (create it in the same folder)

```bash
# Required to use the CurseForge API**
CF_API_KEY=your_curseforge_api_key_escaped

# NOTE: if your key contains a `$` you must escape it as `$$`
# (e.g. $$11$$22...)
```

You can obtain a free CurseForge API key here: 👉 https://console.curseforge.com

# 🧩 4. Adding / Removing / Updating Mods

#### ✅ Add a mod manually
- Download the `.jar` from the CurseForge site.
- Place it inside `./data/mods/`.
- Restart the container:

```bash
docker compose restart mc
```

> ⚠️ Use mods that are compatible with the modloader (Forge / Fabric) chosen by the modpack.

#### ❌ Remove a mod
- Delete the corresponding `.jar` from `./data/mods/`.
- (Optional) Delete any related configuration files in `./data/config/`.
- Restart the container.

#### 🔄 Switch to a completely different modpack
- Change `CF_SLUG` (e.g., from `better-mc-forge-bmc4` to `create-above-and-beyond`) in `docker‑compose.yml`.
- Recommended: either delete or move the existing ./data/ folder.
- Launch the new server:

```bash
docker compose down
mv data data_bak
docker compose up -d
```

#### 🧼 5. Safe Cleanup Script (`clean‑mc.sh`)
A small Bash script you can run on the host VM to keep the server tidy:

```bash
#!/bin/bash

DATA_DIR="./data"
LOG_DIR="$DATA_DIR/logs"
BACKUPS_DIR="$DATA_DIR/backups"

echo "[*] Cleaning logs and temporary files..."

# Delete logs older than 7 days
find "$LOG_DIR" -name "*.log" -type f -mtime +7 -exec rm -v {} \;

# Delete crash reports older than 14 days
find "$DATA_DIR" -name "crash-*.txt" -type f -mtime +14 -exec rm -v {} \;

# Delete automatic backups older than 10 days
if [ -d "$BACKUPS_DIR" ]; then
    find "$BACKUPS_DIR" -type f -mtime +10 -exec rm -v {} \;
fi

# Delete disabled mods (*.disabled)
find "$DATA_DIR/mods" -name "*.disabled" -exec rm -v {} \;

echo "[✓] Cleanup completed."
```

#### ✅ Automatic execution (cron)
Schedule it weekly with cron:

```bash
crontab -e
```

Add the following line (runs every Sunday at 03:00 am):

```bash
0 3 * * 0 /path/to/clean-mc.sh >> /var/log/minecraft-clean.log 2>&1
```

##### 🔐 Recommended Security Measures

```nash
# Set proper ownership
chown -R $(whoami):$(whoami) ./data

# Restrict script permissions
chmod 700 clean-mc.sh
```

If you want to prevent modifications via the web UI, mount the volume as read‑only for the `mods/` or `config/` directories.

That’s it! You now have a fully functional, mod‑enabled Minecraft dedicated server running in Docker, plus the tools to add/remove mods, switch modpacks, and keep the environment clean and secure. 🎉
