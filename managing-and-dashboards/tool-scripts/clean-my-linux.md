#### 🔧 Script Goals:
- Clean APT cache and temporary files
- Remove orphaned packages
- Delete obsolete logs
- Empty the trash bins
- Prune unused Docker images/volumes (if Docker is installed)
- Clean up journalctl logs
- Remove old, unused kernels (safely)


#### 📜 Script: cleanmylinux.sh
```bash
#!/bin/bash

# CleanMyLinux by nuggetz — version 1.0
# Compatible with Debian/Ubuntu
# Run with sudo

echo "🔍 Starting system cleanup..."

# Permission check
if [[ $EUID -ne 0 ]]; then
  echo "❌ This script must be run as root. Use sudo."
  exit 1
fi

# Colored output
info() { echo -e "\033[1;34mℹ️  $1\033[0m"; }
success() { echo -e "\033[1;32m✅ $1\033[0m"; }
warn() { echo -e "\033[1;33m⚠️  $1\033[0m"; }

# 1. APT cache cleanup
info "Cleaning APT cache..."
apt clean && apt autoclean
success "APT cache cleaned."

# 2. Remove orphaned packages
info "Removing orphaned packages..."
apt autoremove -y
success "Orphaned packages removed."

# 3. Clean /tmp and user cache
info "Cleaning /tmp and user cache..."
rm -rf /tmp/* 2>/dev/null
rm -rf ~/.cache/thumbnails/* 2>/dev/null
success "Temporary directories cleaned."

# 4. Remove obsolete logs
info "Cleaning old log files..."
find /var/log -type f -name "*.gz" -delete
find /var/log -type f -name "*.1" -delete
journalctl --vacuum-time=7d
success "Old logs deleted."

# 5. Empty user trash bins
info "Emptying user trash bins..."
for d in /home/*/.local/share/Trash /root/.local/share/Trash; do
  if [ -d "$d/files" ]; then
    rm -rf "$d/files/"*
    rm -rf "$d/info/"*
  fi
done
success "Trash bins emptied."

# 6. Docker cleanup (if installed)
if command -v docker &> /dev/null; then
  info "Pruning Docker (unused images, volumes, cache)..."
  docker system prune -af --volumes
  success "Docker cleaned."
else
  warn "Docker not installed. Skipping."
fi

# 7. Snap cleanup (if installed)
if command -v snap &> /dev/null; then
  info "Removing old Snap revisions..."
  snap list --all | awk '/disabled/{print $1, $3}' | while read snapname revision; do
    snap remove "$snapname" --revision="$revision"
  done
  success "Old Snap revisions removed."
else
  warn "Snap not installed. Skipping."
fi

# 8. Remove old kernels (except current)
info "Removing unused old kernels (if any)..."
CURRENT_KERNEL=$(uname -r)
dpkg -l | grep linux-image | awk '{ print $2 }' | grep -v "$CURRENT_KERNEL" | xargs apt purge -y
success "Old kernels removed."

info "✅ Cleanup complete!"

# Final disk usage
info "📊 Free space after cleanup:"
df -h /
```

#### 🧪 How to Use It:
- Save the script:
```bash
nano cleanmylinux.sh
```
- Paste the code, then save and exit.
- Make it executable:
```bash
chmod +x cleanmylinux.sh
```
- Run with root privileges:
```bash
sudo ./cleanmylinux.sh
```
