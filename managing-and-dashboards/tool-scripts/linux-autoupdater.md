### 🧩 Steps Performed by the Script

* Optional shutdown of Docker containers (if present)
* Full system update via `apt-get`
* Cleanup (`autoremove`, `clean`)
* Prompt for reboot
* Logging to a dedicated file

---

### 🛠️ Script: `update_vm.sh`

```bash
#!/bin/bash
# =========================================================
# Safe update script for VMs running Docker
# Author: nuggetz
# Usage: manual only, no automatic execution
# =========================================================

set -euo pipefail

# === Variables
LOGFILE="/var/log/update_vm.log"
NOW=$(date "+%Y-%m-%d %H:%M:%S")
SEPARATOR="============================================================"

# === Logging start
echo -e "\n$SEPARATOR" | tee -a "$LOGFILE"
echo "🔧 [$NOW] Starting system update..." | tee -a "$LOGFILE"
echo "$SEPARATOR" | tee -a "$LOGFILE"

# === 1. Stop Docker (if installed)
if command -v docker &>/dev/null; then
    echo "⛔ Stopping running Docker containers..." | tee -a "$LOGFILE"
    docker ps -q | xargs -r docker stop | tee -a "$LOGFILE"
else
    echo "⚠️ Docker not found. Skipping container shutdown." | tee -a "$LOGFILE"
fi

# === 2. Perform APT system update
echo "📦 Running APT update and upgrade..." | tee -a "$LOGFILE"
sudo apt-get update | tee -a "$LOGFILE"
sudo apt-get upgrade -y | tee -a "$LOGFILE"
sudo apt-get full-upgrade -y | tee -a "$LOGFILE"
sudo apt-get autoremove -y | tee -a "$LOGFILE"
sudo apt-get clean | tee -a "$LOGFILE"

# === 3. Optional Reboot
echo ""
read -rp "🔁 Do you want to reboot the VM now? (y/N): " confirm
if [[ "$confirm" =~ ^[Yy]$ ]]; then
    echo "♻️ Rebooting..." | tee -a "$LOGFILE"
    reboot
else
    echo "⏭️ Reboot skipped. You can reboot manually later." | tee -a "$LOGFILE"
fi
```

---

### 📦 Installation

Copy the script to each VM:

```bash
sudo nano /home/<user>/tools/update_vm.sh
```

Make it executable:

```bash
sudo chmod +x /home/<user>/tools/update_vm.sh
```

Run it manually whenever you want:

```bash
sudo /home/<user>/tools/update_vm.sh
```

---

### 🔐 Security Considerations

* No `cron` job or automatic triggers included.
* Must be launched **manually by root or via `sudo`**.
* All output is logged in `/var/log/update_vm.log`.
