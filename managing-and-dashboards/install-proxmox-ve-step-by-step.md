# Installing Proxmox VE 🚀

Proxmox VE (Virtual Environment) is a robust open-source platform for virtualization, offering an integrated solution for managing virtual machines and containers with a web-based interface. This guide is enriched with [TechnoTim's](https://technotim.live/posts/first-11-things-proxmox/) precious tips and will walk you through the steps of installing Proxmox VE on your server and fine tuning it to perfection.

## Preliminary: BIOS Settings 🛠️

Before initiating the installation, it's crucial to configure your server's BIOS settings correctly:

- **Boot Mode**: Set to UEFI for newer hardware; otherwise, Legacy mode is acceptable. UEFI is recommended for its modern features and better security. 🖥️
- **Virtualization Technology (VT-x/AMD-V)**: Enable this setting to support hardware virtualization, essential for KVM performance. 🌐
- **Secure Boot**: Disable Secure Boot, as Proxmox's custom kernel might not comply with Secure Boot requirements. 🔒
- **IOMMU (VT-d/AMD IOMMU)**: Enable if you plan to use PCI passthrough features. 📡
- **C-States (CPU power-saving modes)**: Disable C-States to prevent potential performance issues related to variable CPU frequencies. 🚫
- **Intel SpeedStep/AMD Cool'n'Quiet**: Disable these features to ensure consistent CPU performance. ⚡

Make sure to save all changes before exiting the BIOS.

## Prerequisites 📋

- A 64-bit processor with support for Intel VT/AMD-V. 💾
- Minimum 2 GB RAM, with 4 GB or more recommended for production use. 🧠
- At least one Ethernet card. 🌐
- A hard drive with no critical data (the installation will overwrite all data on the disk). 💽
- A bootable installation medium (USB drive) containing the Proxmox VE ISO. 🖥️

## Step 1: Download Proxmox VE ISO 📥

Download the latest Proxmox VE ISO image from the official website:

```url
https://www.proxmox.com/en/downloads
```

Choose the "Proxmox VE ISO Installer" for a complete installation.

## Step 2: Create a Bootable USB Drive Using Balena Etcher 🔄

Balena Etcher is a user-friendly application that allows you to create a bootable USB drive quickly and reliably.

1. Download and install Balena Etcher from [https://www.balena.io/etcher/](https://www.balena.io/etcher/).
2. Open Balena Etcher and click "Flash from file" to select your downloaded Proxmox VE ISO.
3. Insert your USB drive and select it in the "Select target" step.
4. Click "Flash!" to start the process. Once done, your USB drive will be ready to use. 🎉

## Step 3: Install Proxmox VE 🛠️

1. Insert the bootable USB drive into your server and reboot.
2. Boot from the USB drive and select "Install Proxmox VE" from the boot menu.
3. Follow the installer prompts, setting up your disk and decide if you what to RAID the VE storage, location, network, and root password.

## Step 4: Accessing the Proxmox Web Interface 🌐

Once the installation is complete, access the Proxmox web interface via:

```plaintext
https://your-server-ip:8006
```

Log in using the username `root` and the password you defined during the installation.

## Step 5: After install fine tuning 👨‍🔧

### Enable Updates:

Proxmox Virtual Environment 8.1.4 by default every night as an update schedule that pulls down fro predetermined repositories the necessary files.
If you don't own a subscription plan it's ok but we have to make some changes.

1. Go to `Updates` > `Repositories`
2. In the part that says "File: /ect/apt/sources.list (3 Repositories)" `Add` **NO-Subscription**
3. In the part that says "File: /etc/apt/sources.list.d/ceph.list (1 Repository)" `Add` **Ceph Quincy No-Subscription** and **Ceph Reef No-Subscription**
4. Then go to `Updates` and hit `Refresh`
5. It's addvisable to do a reboot
 
### Preparing your storage:

1. Go to your-server-name in the top left of the screen
2. Then click on `disks`
3. Note that you have an **LVM** disk which is your Proxmox installation disk and then all the other might be partitioned or formatted in some other way. Write down the device name of the disks you want to wipe and prepare
4. **Be carefull because the next steps will wipe all yor data and initialise your storage disks**
5. `SSH` in to your server
6. Then run:

```bash
# replace sda with the device name of your choice
fdisk /dev/sda
```

7. Then P for partition, then D for delete, then W for write. Do this for every disk you want to wipe
8. Then Go to `disks` > `ZFW` on the Proxmox dashboard
9. Then click on **create ZFS**
10. Chose a name for the ZFS and a RAID level ([Click HERE to see a descrition of all RAID types]([https://en.wikipedia.org/wiki/Standard_RAID_levels]))
11. Leave `compression` **ON** and `ashift` at **12** (as recommended by Proxmox)
12. Then select all your desired disks and click on **Create**

### Check SMART Monitoring:

By default, smartmontools daemon smartd is active and enabled, and scans the disks under /dev/sdX and /dev/hdX every 30 minutes for errors and warnings, and sends an e-mail to root if it detects a problem. To check if you did not disabled it by accident:

1. Go to your-server-name in the top left of the screen
2. Then click on `disks`
3. Check if every disk has the `PASSED` value under the `S.M.A.R.T.` column

If you want to check it using the terminal run:

```bash
# replace sda with the device name of your choice
smartctl -a /dev/sda
```

### IOMMU (PCI Passthrough):

IOMMU (Input/Output Memory Management Unit) is a feature of modern CPUs/Motherboards that allows the operating system to map physical and virtual memory addresses to manage resources efficiently. This will enable the phisic hardware passthrough of your VMs.

❗️ If see `No IOMMU detected, please activate it. See Documentation for further information.` It means that IOMMU is not enabled in your BIOS or that it has not been enabled in Proxmox yet. If you’re seeing this and you’ve enabled it in your BIOS, you can enable it in Proxmox below.❗️

1. Firstly you will want to check if Vt-d / IOMMU is enabled in your BIOS. Pleas not that every manufacturer could name this feature in a different manner, so check on the documentation of your HW how to enable virtualization
2. Then you will want to check what type of boot manager do you have:

```bash
efibootmgr -v
```

  If it returns an errors, it’s running in Legacy/BIOS with GRUB, skip to **GRUB section**

```bash
Boot0002* proxmox	HD(2,GPT,b0f10348-020c-4bd6-b002-dc80edcf1899,0x800,0x100000)/File(\EFI\proxmox\shimx64.efi)
```

  if it returns something like this down here, it’s running `system-boot`, skip to **system-d** section section
  
```bash
Boot0006 * Linux Boot Manager [...] File(EFI\systemd\systemd-bootx64.efi)
```

  ### GRUB:

  If you’re using GRUB, use the following commands:

  ```bash
  nano /etc/default/grub
  ```

  add `iommu=pt` to `GRUB_CMDLINE_LINUX_DEFAULT` like so:

  ```bash
  GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"
  ```

  If you aren’t using an intel processor, remove `intel_iommu=on`

  ### system-boot:

  If you’re using system-boot use the following commands:

  ```bash
  nano /etc/kernel/cmdline
  ```

  add `intel_iommu=on iommu=pt` to the end of this line without line breaks

  ```bash
  root=ZFS=rpool/ROOT/pve-1 boot=zfs intel_iommu=on iommu=pt
  ```

  If you aren’t using an intel processor, remove `intel_iommu=on`

  then run:

  ```bash
  pve-efiboot-tool refresh
  ```

  and:

  ```bash
  reboot
  ```

### VFIO modules:

The VFIO driver framework provides unified APIs for direct device access. It is an IOMMU/device-agnostic framework for exposing direct device access to user space in a secure, IOMMU-protected environment.

1. Edit this file running:

```bash
nano /etc/modules
```

2. Add these few lines:

```bash
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```

3. Then run:

```bash
update-initramfs -u -k all
```

4. And finally:

```bash
reboot
```

### Loading VFIO modules:

To ensure proper GPU passthrough of the integrated GPU (iGPU) to your VMs or containers, verify that the Intel i915 driver is loaded in the server’s kernel by running the following command:

```bash
lsmod | grep i915
```

### Next Step: Configure iGPU Passthrough

We now need to:
- Check the IOMMU group of the integrated GPU (i915)
This is important to determine whether the iGPU is isolated in its own IOMMU group, a requirement for proper passthrough.
- Configure the VM to passthrough the iGPU using the /dev/dri device
For Intel iGPUs, it’s common to passthrough the DRI (Direct Rendering Infrastructure) device rather than using direct PCI passthrough.

1. Check the IOMMU Group for the Intel iGPU

Run the following command:

```bash
find /sys/kernel/iommu_groups/ -type l | grep 00:02.0
```

00:02.0 is the PCI address of your Intel iGPU (you can confirm it using lspci).

If you see output like:

```bash
/sys/kernel/iommu_groups/xx/devices/0000:00:02.0
```

(where xx is a number), it indicates the IOMMU group to which the iGPU belongs.


2. Confirm That the IOMMU Group Contains Only the iGPU

To list all devices within the same group:

```bash
ls -l /sys/kernel/iommu_groups/xx/devices/
```

Replace xx with the group number found in the previous step.

- If the group contains only the iGPU (00:02.0), you’re good to go for passthrough.
- If there are other critical devices in the same group, passthrough may cause stability or security issues.

### Next Step: Bind the Intel GPU to the vfio-pci Driver

- First, identify the vendor:device ID of the integrated GPU (Even though you already have it, let’s double-check for clarity):

```bash
lspci -nn | grep 00:02.0
```

This should return something like:

```bash
00:02.0 VGA compatible controller [0300]: Intel Corporation HD Graphics 530 [8086:1912] (rev 06)
```
So the vendor:device ID is 8086:1912.


- Create (or edit) the /etc/modprobe.d/vfio.conf file

This tells vfio-pci to handle the device:

```bash
echo "options vfio-pci ids=8086:1912" > /etc/modprobe.d/vfio.conf
```

- Blacklist the default i915 driver

To avoid conflicts by preventing the Intel driver from loading:

```bash
echo "blacklist i915" > /etc/modprobe.d/blacklist-i915.conf
```

- Rebuild the initramfs

This applies the changes so they’re effective at boot:

```bash
update-initramfs -u
```

- Reboot the Proxmox host
```nash
sudo reboot
```

After Reboot: Verify That vfio-pci Is Now Managing the GPU

Run:

```bash
lspci -k -nn | grep -A 3 8086:1912
```

You should see an output similar to:

```
Kernel driver in use: vfio-pci
```

### VLAN Awareness:

A VLAN-aware device is the one which understands VLAN memberships and VLAN formats. If you want your Proxmoxx to be VLAN Aware:

1. Go to your-server-name in the top left of the screen
2. Then go to `Network`
3. Select your network bridge
4. Check the box `VLAN aware`
5. Click `OK`

If you want then to restrict **ALL VMs** to a certain VLAN:

1. Run:

```bash
nano /etc/network/interfaces
```

2. After `bridge-vids` put the name of the vlan tha you want

```bash
bridge-vlan-aware yes
bridge-vids 20
```

If you want to restrict **A certain VM** to a **specific VLAN**:

1. Go under your-server-name in the top left of the screen
2. Select your choosen VM
3. Select `Hardware`
4. Then `Networc Device`
5. And set the VLAN in the `VLAN tag` dropdown menu
6. Click `OK`

## Step 5: After install fine tuning 💾

To put in place a scheduled backup job to save your VMs in case of need:

1. Go to Datacenter in the top left of the screen
2. Then click on `Backups` > `Add`
3. In `General` choose the schedule time frame, the included VMs and if you want to send an e-mail at the end of the backup
4. Finally click on `Create`

## Conclusion 🎊

You're now ready to use Proxmox VE! Begin creating virtual machines, containers, and exploring Proxmox's vast features. Consult the Proxmox documentation for more in-depth information:

```url
https://pve.proxmox.com/pve-docs/
```
