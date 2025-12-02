# 📺 The Ultimate VM: Servarr Media Server

This guide walks you through the creation and configuration of the ultimate virtual machine, a media server running the Servarr suite on your home Proxmox setup.

---

## 💻 Host System Specs

My home server is based on:

```bash
Model: HP Elitedesk 800 G5 SFF
CPU: Intel i5-6500 @ 3.2 GHz
RAM: 64 GB DDR4
```

> If you have at least this set up you are good to go.

---

## 🛠️ Step 1: Create the Virtual Machine

Follow these steps from the **Proxmox Web GUI**:
- Open the **Proxmox dashboard** and click **“Create VM”**.
- Enter the VM **name**.
- Under Advanced, enable **✅ Start at boot**.
- Click **Next**.

⸻

#### 📀 OS Tab
- Choose an ISO image for **Ubuntu Server** from your storage.
- Click **Next**.

⸻

#### ⚙️ System Tab
- Change **Machine** to q35.
- Change Firmwar to **OVMF (UEFI)**
- Selet a disk to mount the UEFI Partition
- Click **Next**.

⸻

#### 💾 Hard Disk Tab
- Allocate **64 GB** of disk space.
- Under **Advanced Options**:
	- If you’re using an NVMe drive, check **✅ SSD Emulation**.
	- Otherwise, leave it unchecked.
- Click **Next**.

⸻

#### 🧠 CPU Tab
- Choose the number of cores you want to assign.
- In my case: **1 socket, 4 cores**.
- Set **CPU Type** to host (for performance passthrough).
- Click **Next**.

⸻

#### 🧬 Memory Tab
- Allocate RAM for the VM.
- I used **16 GB** → enter 16384 MB.
- Click **Next**.

⸻

#### 🌐 Network Tab
- Leave most defaults.
- If you’re using VLANs (like I am), enter the appropriate **VLAN Tag**.
- This ensures the VM attaches to the correct virtual LAN segment.
- Click **Next** and finish VM creation.

⸻

#### 🚀 Set the Graphics Passthrought for later HW Accelleration
- Go to the newly created VM, under "**Hardware**"
- Click **Add** and select **PCI** Device
- Select RAW Device and in the list you sould see your isolated VirtIO card (set in the proxmox post-install procedures)
- In the "**Advanced**" settings of the PCI:
	- **All Functions: Do not check this option** — it’s not needed for an iGPU. This is only required for discrete GPUs (e.g., Nvidia) that have multiple functions such as audio and video on separate interfaces.
	- **ROM-Bar: Leave this enabled** (default setting). It should remain checked.
	- **Primary GPU**: **Check this box only if you want to perform a full passthrough** of the main display output (e.g., direct video output to a monitor via the VM).

---

## 📀 Step 2: Install ubuntu server
- Start the **VM**.
- Open a **console** and proceed with the standard guided installation.

---

## 🛠️ Step 3: Post-Installation Preparation — Getting Ready for Docker & Automation

Now that your VM is up and running, it’s time to **prepare the file system structure** and lay the groundwork for what comes next:
🚀 the installation of Docker and the entire **Servarr stack** for your automated media server.

We’re also going to create a set of helper scripts 🧰 that will make it super easy to **update your containers and your system in the future**, with just a single command. This is all about building a clean, maintainable setup from the very start. Let’s go!

---

#### Confirm some basic facts about the VM and the GPU passthrough:

- Check for presence and status of the passthrough GPU inside the VM

Run inside the VM:

```bash
lspci -nnk | grep -A 3 8086:1912
```

You should see the Intel iGPU (ID 8086:1912) and which driver is currently in use.

⸻

Drivers and modules
	- If the VM is Linux (e.g., Ubuntu), make sure the i915 driver is loaded and functioning:
```bash
lsmod | grep i915
```
Also check for errors with:

```bash
dmesg | grep i915
```

⸻

#### Verify the GPU framebuffer is active (useful for GUI and passthrough)

```bash
glxinfo | grep "OpenGL renderer"
```

(If the VM is SSH-only, you can skip this step or install necessary packages.)

---

#### Known Errors

The messages indicate that the **i915 driver cannot locate the video BIOS data** required to correctly configure the video outputs and display controllers (CRTC).

* “Cannot find any crtc or sizes”**

- This error is typical in **Intel integrated‑GPU passthrough** setups, especially when **no physical video output is connected**.  
- It means the driver cannot detect an active display or set up a video framebuffer.  
- If the VM is **headless** (no GUI, server‑only), this isn’t necessarily a problem because you don’t need a direct graphical output.  
- However, it can affect features that rely on an active framebuffer (e.g., some OpenGL applications).

**`glxinfo` fails because there is no active X display**

- On a headless VM **without an X server or Wayland compositor**, `glxinfo` will fail because there is **no graphics environment**.  
- This behavior is expected and normal.

---  
**What This Means for Your Use Case**

- You want to use the **Intel iGPU via passthrough for Jellyfin**, which will employ **hardware acceleration through VAAPI / Intel Quick Sync** for video transcoding inside Docker containers.  
- You **don’t need a direct video output, GUI, or desktop environment**.  
- The only requirement is that the **iGPU device works for hardware acceleration**; a missing framebuffer or X server does **not** impact Jellyfin’s transcoding tasks.

---

#### What’s Next

**Verify the DRM and VAAPI devices that are available inside the VM**

- Check that the DRM device exists:

```bash
ls -l /dev/dri
```

You should see at least `card0` and `renderD128`.
If they are present, the DRM driver is active.

- Install the packages required for VAAPI and Intel Quick Sync Video (QSV):
  
```Bash
sudo apt update
sudo apt install vainfo intel-media-va-driver-non-free
```

`vainfo` lets you see whether the system recognises VAAPI acceleration.
- Run `vainfo`:

```bash
vainfo
```

The warnings and errors you may see from the i915 driver when the Intel iGPU is passed through are normal and **non‑blocking** as long as you are not using a direct video output—especially in headless environments.
The important thing is that the DRM devices `/dev/dri/card0` and `/dev/dri/renderD128` are present and that VAAPI works correctly for Jellyfin.

> **Why `vainfo` may fail:** In a server/headless setup `vainfo` tries to connect to an X11 display, which does not exist. This failure is expected and harmless.

---

#### Completed Checks:

| **Component** | **Status** | **Notes** |
|-------------------------------------|---|--------------------------------------|
| i915 driver loaded | ✅ | No notes |
| `/dev/dri/renderD128` support | ✅ | Correct device for headless GPU passthrough |
| i915 and `card0` visible in the VM | ✅ | No notes |
| `vainfo` fails | ⚠️ | Normal – no X server available |

---

#### Next Steps

**Verify VA-API support from the terminal**

- install the tools that **do not require a GUI:**

```bash
sudo apt install -y intel-gpu-tools vainfo libva-drm2 libva2
```

- Test VAAPI directly against the DRM backend (no X11):

```bash
LIBVA_DRIVER_NAME=i965 vainfo --display drm
# or
LIBVA_DRIVER_NAME=iHD vainfo --display drm
```

If either command succeeds you’ll see output similar to:

```bash
vainfo: VA-API version: ...
vainfo: Driver version: ...
vainfo: Supported profile and entrypoints
```

**Install the full Intel VAAPI driver stack**

```bash
sudo apt install -y intel-media-va-driver-non-free i965-va-driver libva-drm2 libva2 vainfos
```

- Retest:

```bash
LIBVA_DRIVER_NAME=i965 vainfo --display drm
LIBVA_DRIVER_NAME=iHD  vainfo --display drm
```

Successful output will list the supported profiles (H.264, HEVC, VP9, etc.).

**[Optional] Create a virtual framebuffer with Xvfb (only useful for testing vainfo, not required for Jellyfin)**

```bash
sudo apt install -y xvfb
Xvfb :0 &
export DISPLAY=:0
vainfo
```
This spins up a dummy X server so vainfo can run, but you should not use this in production for Jellyfin.

---

##### Verify permissions on `/dev/dri/*`

```bash
ls -l /dev/dri
```

- Typical output:

```bash
crw-rw---- 1 root render ...
```

- If your user is not a member of the render group, add it:

```bash
sudo usermod -aG render $(whoami)
newgrp render   # refresh group membership in the current session
```

After adjusting the group, run vainfo again to confirm it works.

---

## Step 3: 📁 Create the Folder Structure

Let’s create the folders that will host your services and your management scripts.

```bash
mkdir -p /your-path/services
mkdir -p /your-path/tools/update-services
mkdir -p /your-path/services/qbittorrent/config
mkdir -p /your-path/services/prowlarr/config
mkdir -p /your-path/services/radarr/config
mkdir -p /your-path/services/sonarr/config
mkdir -p /your-path/services/lidarr/config
mkdir -p /your-path/services/readarr/config
mkdir -p /your-path/services/jellyfin/config
mkdir -p /your-path/services/bazarr/config
mkdir -p /your-path/services/jellyseerr/config
```

and ensure that your user has the rights to do whatever it wants:

```bash
chown -R 1000:1000 /your-path/services
```

#### 📂 Move into the Working Directory

Now move into the directory where we’ll drop the Docker Compose file:

```bash
cd /your-path/tools/update-services
```

#### 🧾 Create the docker-compose.yml File

- This file will define all the containers needed to run your automated media server (e.g., Radarr, Sonarr, Lidarr, etc.).

```bash
sudo nano docker-compose.yml
```

- Paste all the docker compose `yml` from the servar apps documentation. I know, we have no docker installed yet... but we will already be ready when docker will be installed. It really depends on what apps do you wish touse, but here an example of my test configuration:

```yml
services:
  portainer_agent:
    container_name: portainer_agent
    image: portainer/agent:lts
    restart: always
    ports:
      - "9001:9001"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/docker/volumes:/var/lib/docker/volumes
      
  qbittorrent:
    image: ghcr.io/hotio/qbittorrent
    container_name: qbittorrent
    ports:
      - "8080:8080"
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=Europe/Rome
      - WEBUI_PORTS=8080/tcp,8080/udp
    volumes:
      - /home/user/services/qbittorrent/config:/config
      - /smb/data:/data
    restart: unless-stopped

  prowlarr:
    image: ghcr.io/hotio/prowlarr
    container_name: prowlarr
    ports:
      - "9696:9696"
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=Europe/Rome
    volumes:
      - /home/user/services/prowlarr/config:/config
    restart: unless-stopped

  radarr:
    image: ghcr.io/hotio/radarr
    container_name: radarr
    ports:
      - "7878:7878"
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=Europe/Rome
    volumes:
      - /home/user/services/radarr/config:/config
      - /smb/data:/data
    restart: unless-stopped

  sonarr:
    image: ghcr.io/hotio/sonarr
    container_name: sonarr
    ports:
      - "8989:8989"
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=Europe/Rome
    volumes:
      - /home/user/services/sonarr/config:/config
      - /smb/data:/data
    restart: unless-stopped

  lidarr:
    image: ghcr.io/hotio/lidarr
    container_name: lidarr
    ports:
      - "8686:8686"
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=Europe/Rome
    volumes:
      - /home/user/services/lidarr/config:/config
      - /smb/data:/data
    restart: unless-stopped

  readarr:
    image: ghcr.io/hotio/readarr
    container_name: readarr
    ports:
      - "8787:8787"
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=Europe/Rome
    volumes:
      - /home/user/services/readarr/config:/config
      - /smb/data:/data
    restart: unless-stopped

  jellyfin:
    image: ghcr.io/hotio/jellyfin
    container_name: jellyfin
    ports:
      - "8096:8096"
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=Europe/Rome
      - LIBVA_DRIVER_NAME=iHD   # solo se usi Intel
    devices:
      - /dev/dri:/dev/dri
    group_add:
      - "993"
    volumes:
      - /home/user/services/jellyfin/config:/config
      - /smb/data/media:/data
    restart: unless-stopped

  bazarr:
    image: lscr.io/linuxserver/bazarr:latest
    container_name: bazarr
    ports:
      - "6767:6767"
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Rome
    volumes:
      - /home/user/services/bazarr/config:/config
      - /smb/data/media:/data
    restart: unless-stopped

  jellyseerr:
    image: ghcr.io/hotio/jellyseerr
    container_name: jellyseerr
    ports:
      - "5055:5055"
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=Europe/Rome
    volumes:
      - /home/user/services/jellyseerr/config:/config
    restart: unless-stopped

  flaresolverr:
    image: ghcr.io/flaresolverr/flaresolverr:latest
    container_name: flaresolverr
    ports:
      - "8191:8191"
    environment:
      - LOG_LEVEL=info
    restart: unless-stopped
```

- 💾 Then press CTRL + X, then Y, then ENTER to save.
  
> **NOTE THIS:** I mapped all the data for the media apps in a **non-existant** /smb folder. **THIS WILL BE IMPORTANT LATER** so choose a name for an external folder that will be mounted to this V;. I choose to call it "smb" because will be a NAS over the network that will contain all my media files such as movies or other stuff. In this guide wi will be attaching a drive internali but the procedure is exactly the same.

**NOTE ABOUT JELLYFIN**
- devices exposes the real /dev/dri device to the container.
- group_add: "993" adds the container to the **render** group (actually GID 993, which represents the render group on your host).
- Even though the container runs with UID/GID 1000, access to /dev/dri is granted because the process inside the container is also a member of group 993, which has the necessary permissions on that device.

## 🧱 Step 4: Create a ZFS Pool with Mirrored HDDs (via Proxmox Web GUI)

We’re now going to **create a ZFS pool** using two hard drives in **mirror mode**. This will be the main **data volume** for your VM, where all the Servarr media files will be stored — with redundancy for peace of mind.

⚠️ Note: These disks **will be wiped** during pool creation. Be sure they are empty or backed up.

#### 🌍 Log Into the Proxmox Web Interface

- Open your browser and go to the Proxmox IP or hostname (e.g., https://10.0.0.100:8006).
- Login with your Proxmox credentials.

⸻

#### 🏠 Open the Datacenter View

- Click on “**Datacenter**” in the left sidebar to ensure you’re working at the cluster level.
- From here you can manage all storage devices and pools.

⸻

#### 💾 Navigate to Disk Management

- Click on the node where your disks are installed.
- Go to the “**Disks**” section in the left sidebar.
- Here you’ll see all physical disks (SSDs, HDDs, NVMe, etc.).

⸻

#### 🔍 Identify the Disks for the Mirror

- Look for the **two unused HDDs** you want to use (e.g., /dev/sda and /dev/sdb).
- You can recognize them by size, model, or if they are listed as “not in use”.

✏️ **Write down** their device paths – you’ll need them during pool creation.

⸻

#### ✨ Create the ZFS Mirror

- Go to the “**ZFS**” menu on the left, then click “**Create ZFS**”.
- In the dialog:
- **RAID Level**: Choose Mirror
- **Disks**: Select the two previously identified HDDs
- **Name**: Set a meaningful name like storage0 or media-pool
- Other settings (ashift, compression) can be left to defaults unless you have special needs.
- Click **Create**. Proxmox will wipe the drives and initialize the ZFS pool.

✅ You now have a **redundant, high-performance storage pool** ready to use.

---

## 🔗 Step 5: Attach the ZFS Pool as Storage for Your VM

We’ll now create a **virtual disk** inside the new ZFS pool and attach it to the VM dionysus, so that it can be formatted and used inside the OS.

#### 🖇️ Attach ZFS Volume to VM

- In the sidebar, find your VM (dionysus) and click on it.
- Go to the “**Hardware**” tab.
- Click “**Add**” → “**Hard Disk**”.
	- **Bus/Device**: VirtIO Block (recommended for performance)
	- **Storage**: Select your ZFS pool (storage0)
	- **Disk Size**: Allocate the space you want (e.g., 1 TB or full available)
	- **Cache**: Write back (unsafe) is fastest but Default is safest
	- Click **Add** to attach the disk.

🔁 The new disk will now appear as /dev/sdb (or similar) in the guest OS.

---

## 🧰 Step 6: Preparing the Disk in Linux (Inside the VM)

Once attached, we need to format and mount the new disk inside your VM’s OS.

⸻

#### 👨‍💻 Access the VM Console

- From Proxmox, click “**Console**” on the dionysus VM, or SSH in if you prefer.

⸻

#### 📋 Partition, Format and Mount the Disk

- Identify the new disk:

```bash
lsblk
```

- Format it with ext4:

```bash
sudo mkfs.ext4 /dev/sdb
```

📝 You can also use xfs or btrfs if preferred, but ext4 is a solid, stable default.

- Create a mount point (for example /smb, because in my case I will later export it via Samba):

```bash
sudo mkdir /smb
```

- Mount the disk:

```bash
sudo mount /dev/sdb /smb
```

> At this point, the disk is ready to use — but not yet persistent across reboots.

---

## 🔁 Step 7: Make the Mount Persistent with fstab

To make sure the disk mounts automatically at every boot, we need to edit `/etc/fstab`.

#### 📛 Get the UUID of the New Disk

```bash
sudo blkid /dev/sdb
```

Sample output:

```bash
/dev/sdb: UUID="1234-ABCD" TYPE="ext4"
```

⸻

#### 📝 Edit /etc/fstab

```bash
sudo nano /etc/fstab
```

Add this line at the bottom, replacing the UUID with the one from above:

```bash
UUID=1234-ABCD   /smb   ext4   defaults,noatime   0 2
```
- defaults,noatime: basic mount options with reduced disk wear.
- 0 2: standard `dump/fsck` flags.

---

#### 🧪 Test the Configuration

Before rebooting, verify the entry is correct:

```bash
sudo mount -a
```

> If no errors appear, you’re good to go!

---

## Step 8: 📁 Create Directory Structure

Once /smb is mounted, create the required folder hierarchy:

```bash
sudo mkdir -p /smb/data/{torrents/{books,movies,music,tv},usenet/{incomplete,complete/{books,movies,music,tv}},media/{books,movies,music,tv}}
sudo chown -R 1000:1000 /smb
sudo chmod -R 775 /smb
```

**🔍 Explanation:**
- Uses Bash brace expansion to efficiently create nested directories.
- Organizes downloads and media by category and media type for clarity and easy management.

---

## Step 9: 🐳 Install Docker

Docker enables lightweight, isolated containerized applications. To install Docker, run:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
```

⚠️ **Important**: Log out and back in to apply the docker group membership changes.

✅ **Verify Docker installation:**

```bash
docker version
```

🎯 **Test Docker functionality:**

```bash
docker run hello-world
```

- Runs a test container that confirms Docker is installed and working properly.

## Step 10: 📦 Add Docker APT Repository & Install Docker Compose Plugin (Recommended)

#### 1️⃣ Setup Docker’s official APT repository:

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
```

#### 2️⃣ Install Docker Compose plugin:

```bash
sudo apt-get install docker-compose-plugin
```

#### 3️⃣ Verify Compose installation:

```bash
docker compose version
```

✅ Modern Docker Compose integrates as a plugin to Docker CLI, replacing the older standalone binary.

---

## Step 11: 🐳 Ensure Docker Starts Only After /smb is Mounted

It’s **critical**: if Docker starts **before the disk is mounted**, any bind-mount will fail and containers relying on /smb will break.
We’ll solve this cleanly with a **custom systemd service**.

**🧩 Goal**
- ⏳ **Delay Docker startup** until /smb is mounted.
- ✅ Clean solution using systemd on Linux (e.g., Ubuntu Server, Debian).


#### 🔍 Check if Docker Uses systemd

Ensure your Docker daemon is managed by systemd:

```bash
systemctl status docker
```

✅ You should see output like:

```bash
Loaded: loaded (/lib/systemd/system/docker.service; enabled; ...)
```

⸻

#### 🛠️ Create Docker Wait Service

This service ensures Docker starts only after /smb is mounted.

📄 Create the file:

```bash
sudo nano /etc/systemd/system/docker-wait-smb.service
```

✍️ Paste the following content:

```bash
[Unit]
Description=Start Docker only after /smb is mounted
Requires=local-fs.target
After=local-fs.target

[Service]
Type=oneshot
ExecStart=/bin/bash -c 'while ! mountpoint -q /smb; do sleep 1; done'
RemainAfterExit=true

[Install]
WantedBy=docker.service
```

💡 This does not start Docker directly, but waits for /smb to become a valid mountpoint.

⸻

#### 🔗 Link Wait Service to Docker

Now link the wait service to Docker using an override.conf.

📁 Create override config:

```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo nano /etc/systemd/system/docker.service.d/override.conf
```

✍️ Insert:

```bash
[Unit]
Requires=docker-wait-smb.service
After=docker-wait-smb.service
```

⸻

#### 🔄 Reload systemd and Enable the Service

📦 Reload and enable everything:

```bash
# Reload systemd
sudo systemctl daemon-reexec
sudo systemctl daemon-reload

# Enable the wait service
sudo systemctl enable docker-wait-smb.service

# Optional: restart Docker to test
sudo systemctl restart docker
```

⸻

#### ✅ Final Test

1️⃣ Reboot the machine:

```bash
sudo reboot
```

2️⃣ After boot, confirm everything works:

```bash
# Check that /smb is mounted
mount | grep /smb

# Check Docker status
systemctl status docker

# Check the wait service
systemctl status docker-wait-smb
```

⸻

## Step 12: Deploying/Update and Maintenance Scripts and the almighty Start up

This script will allow you to easily **create or refresh** your media containers with a single command. We Will run it shortly to start the whole servarr up and it will also be handy when you need to update the whole thing in one batch.

- Create a file:

```bash
# Change it to be in the path you choose before
sudo nano /your-path/tools/update-services/update-services.sh
```

- Paste this script inside, changeing the parts with the "CHANGE ME" comment.

```bash
#!/bin/bash
# ---------------------------------------------------------
# Author: nuggetz
# ---------------------------------------------------------

set -euo pipefail

LOGFILE="/var/log/update_media_services.log" # CHANGE ME if you want a different name for the logs file
NOW=$(date "+%Y-%m-%d %H:%M:%S")
SEP="============================================================"

log() {
    echo -e "$1" | tee -a "$LOGFILE"
}

log "\n$SEP"
log "🔧 [$NOW] Starting the update procedures..."
log "$SEP"

# ------------------------------------------------------------------
# 1️⃣ Containers shut down.
# ------------------------------------------------------------------
log "⛔ Stopping all Docker containers..."
if docker ps -q >/dev/null; then
    docker stop $(docker ps -q) | tee -a "$LOGFILE"
else
    log "⚪ Nothing to stop."
fi

# ------------------------------------------------------------------
# 2️⃣ Remove all the containers.
# ------------------------------------------------------------------
log "🗑️ Removing all docker containers..."
if docker ps -aq >/dev/null; then
    docker rm -f $(docker ps -aq) | tee -a "$LOGFILE"
else
    log "⚪ Nothing to remove."
fi

# ------------------------------------------------------------------
# 3️⃣ Docker images clean up.
# ------------------------------------------------------------------
log "🧹 Cleaning all Docker images (shared layers and volumes excluded)..."
docker image prune -a -f | tee -a "$LOGFILE"

# ------------------------------------------------------------------
# 4️⃣ Docker Compose procedures.
# ------------------------------------------------------------------
COMPOSE_FILE="/your-path/tools/update-services/docker-compose.yml" # CHANGE ME. Put the path you choose for the docker-compose file

log "⬇️ Pulling images…"
docker compose -f "$COMPOSE_FILE" pull | tee -a "$LOGFILE"

log "▶️ Starting up containers from new images…"
docker compose -f "$COMPOSE_FILE" up -d | tee -a "$LOGFILE"

log "✅ Update completed!"

# ------------------------------------------------------------------
# 5️⃣ Reboot
# ------------------------------------------------------------------
log ""
read -rp "🔁 Do you want to reboot the machine? (y/N): " confirm
if [[ "$confirm" =~ ^[Yy]$ ]]; then
    log "♻️ Rebooting..."
    reboot
else
    log "⏭️ Reboot canceled by the user. You can do it manually later."
fi
```

- Then make it executable:

```bash
sudo chmod +x update-services.sh
```

- do not run it now, wi will be doing it later.

---

#### 🧹 Add a Script to Clean Up Your Linux System

This is a utility script that will take care of general system cleanup (clearing old packages, cleaning logs, etc.).

```bash
cd /your-path/tools
sudo nano cleanmylinux.sh
```
Paste this inside:

```bash
#!/bin/bash

# =========================================================
# Author: nuggetz
# =========================================================

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
Then make it executable:

```bash
sudo chmod +x cleanmylinux.sh
```

- You’ll use this script periodically to **keep your system lean and efficient**.

---

#### 🔄 Add a Script to Keep the OS Updated

This one does a full update of your system — perfect to run once in a while or before rebooting.

```bash
sudo nano /home/dionysus/tools/update-linux.sh
```

Paste this inside:

```bash
#!/bin/bash

# =========================================================
# Safe update script for VMs running Docker
# Author: nuggetz
# Usage: manual only, no automatic execution
# =========================================================

set -euo pipefail

# === Variables
LOGFILE="/var/log/update_vm.log" # CHANGE ME if you need another name for your log files.
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

Make it executable:

```bash
sudo chmod +x update-linux.sh
```

---

## Step 13: 🔄 Running all the scripts and have fun.

- Now that you should all set up, run the scripts in this order:

```bash
sudo /your-path/./update-linux.sh
sudo /your-path/./cleanmylinux.sh
sudo /your-path/./update-services.sh
```

---

## Step 14: Optimizing the Jellyfin app transcoding.

 Your  media server it's ready and you can tell by the portainer dashboard being all lighted in GREEN... hopefully :)
- Now open you Jellyfin app on you tv or device of choice and go to **"Control Panel"** > **"Playback"** > **"Transcoding"**
- Set the Hardware Acceleration to **"Video Acceleration API (VAAPI)"**
- And map the VA-API Device to **"/dev/dri/renderD128"** if not yet automatically mapped

In the audio settings to optimize the audio trancoding
- Set the algorithm to Dave750

---

## GOOD TO GO!

After this infinite guide you are left with all the services and server in place and ready to be configured as your need.
In orther to configure the services i suggest you to follow the official guides here:

- [qBittorent](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/)
- [Prowlarr](https://wiki.servarr.com/prowlarr/quick-start-guide)
- [Sonarr](https://wiki.servarr.com/sonarr/quick-start-guide)
- [Radarr](https://wiki.servarr.com/radarr/quick-start-guide)
- [Lidarr](https://wiki.servarr.com/lidarr/quick-start-guide)
- [Readarr](https://wiki.servarr.com/readarr/quick-start-guide)
- [Bazarr](https://wiki.bazarr.media/)
- [Flaresolverr](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/)

