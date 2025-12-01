# 📺 The ultimate media servarr - Configuration and fine tuning of a dockerized VM (Proxmox)

This guide walks you through the creation and configuration of the ultimate media virtual machine, a media server running the Servarr suite on your home Proxmox server.

⸻

#### 💻 Host System Specs

My home server is based on:
```bash
Model: HP Elitedesk 800 G5 SFF
CPU: Intel i5-6500 @ 3.2 GHz
RAM: 64 GB DDR4
```
If you have at least these specs, you're golden.

⸻

## 🛠️ Step 1: Create the Virtual Machine

Follow these steps from the **Proxmox Web GUI**:
- Open the **Proxmox dashboard** and click **“Create VM”**.
- Enter the VM **name** (e.g., dionysus).
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

⸻

## 📀 Step 2: Install ubuntu server
- Start the **VM**.
- Open a **console** and proceed with the standard guided installation.

⸻

## 🛠️ Step 3: Post-Installation Preparation — Getting Ready for Docker & Automation

Now that your VM is up and running, it’s time to **prepare the file system structure** and lay the groundwork for what comes next:
🚀 the installation of Docker and the entire **Servarr stack** for your automated media server.

We’re also going to create a set of helper scripts 🧰 that will make it super easy to **update your containers and your system in the future**, with just a single command. This is all about building a clean, maintainable setup from the very start. Let’s go!

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

⸻

#### Known Errors

The messages indicate that the i915 driver cannot locate the video BIOS data required to configure the video outputs and display controllers (CRTC).

| **Message** | Explanation** |
|----------------------------------------------------|------------------------------------------------------------------------------|
| **“Cannot find any crtc or sizes”** | Typical for Intel iGPU passthrough when no physical monitor is connected. The driver cannot detect an active display or configure a video framebuffer.|
| **glxinfo fails because there is no active X display** | In a head‑less VM without X or Wayland, `glxinfo` will fail – this is expected. |

#### What This Means for Your Use‑Case
- You intend to use the Intel iGPU **passthrough for Jellyfin**, which will rely on **VAAPI / Intel Quick Sync** for hardware‑accelerated video transcoding inside Docker containers.
- You **do not need a direct video output, GUI, or desktop** – the VM can remain headless.
- The only requirement is that the iGPU device (`/dev/dri/card0` and `/dev/dri/renderD128`) is functional.

#### Verify DRM / VAAPI Devices

```Bash
ls -l /dev/dri
```

You should see at least `card0` and `renderD128`. If they exist, the DRM driver is operational.

#### Install VAAPI Packages

```Bash
sudo apt update
sudo apt install vainfo intel-media-va-driver-non-free
```

Run:

```Bash
vainfo
```

`vainfo` may complain about the lack of an X server; this is normal in a headless environment. The important thing is that the driver reports supported profiles (H.264, HEVC, VP9, etc.).

#### Summary of Checks

| Component | Status | Notes |
|-------------------------------|----|------------------------------------------|
|i915 driver loaded | ✅ | OK |
|/dev/dri/renderD128 present | ✅ | Correct device for headless GPU passthrough |
|vainfo runs (warnings expected)| ⚠️ | Normal – no X server |
|No X server available| ✅ | Expected for a headless Jellyfin host |
