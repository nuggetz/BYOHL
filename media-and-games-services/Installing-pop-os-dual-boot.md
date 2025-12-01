# 📘 INSTALLING POP!_OS ON A SYSTEM WITH NVIDIA GPU

> Note that if you are doing a dual boot on separate drives, is best to disconnect the other disk.

⸻

#### 🛠️ 1️⃣ BIOS CONFIGURATION

Recommended Settings:

**CSM (Compatibility Support):**	Enabled or UEFI Only
**Secure Boot:**	Disabled (recommended)
**Secure Boot Keys:**	Clear all keys
**OS Type:**	Other OS (prevents automatic Windows re-assertion)
**Boot Mode:**	UEFI Only
**Fast Boot:**	Disabled (optional, but helpful during installation)
**SATA Mode	AHCI:** (NOT RAID)
**Primary Display:**	Auto or PCIe

> 👉 Note: If your MOBO is an old ASUS Rampage do not fully disable CSM, otherwise you may lose BIOS access.

⸻

#### 💿 2️⃣ POP!_OS INSTALLATION

**Procedure:**
1.	**Download Pop!_OS** from system76.com/pop. (Use the **NVIDIA version**, specific for dedicated GPUs).
3.	**Create a bootable USB** (using BalenaEtcher or Rufus in GPT/UEFI mode).
4.	**Boot from USB Live**, manually selecting the UEFI USB stick from the BIOS Boot Menu.
5.	At the live boot screen, optionally press **E** on GRUB, then **Ctrl+C** to access an emergency shell if needed.
6.	Launch the installer.
7.	During partitioning:
- Select the **disk dedicated to Pop!_OS**.
- Keep Windows on its own separate disk.
- Use automatic or manual partitioning, creating:
  - EFI partition of 512 MB to 1 GB (if not already present).
  - Root (/) partition and Swap.
  - Pop!_OS will write the Systemd bootloader directly to EFI.
8.	**Proceed with installation** as normal.
9.	Once finished, reboot the system.

⸻

#### 🚨 3️⃣ DUAL BOOT MANAGEMENT (Separate Disks)
- No shared bootloader (GRUB will not interfere with Windows).
- From BIOS, select the disk you want to boot from (Windows or Pop!_OS).
- Pop!_OS will install its UEFI boot entry only on **sda1 (EFI System Partition of Pop!_OS disk).**
 
⸻

#### ⚙️ 4️⃣ POST-INSTALLATION (Fix NVIDIA Error -1)

> ⚠️ If at boot you encounter the error:

```
Probe with NVIDIA driver failed with error -1
```

Follow these steps:
1.	**Boot from the Live USB.**
2.	Open terminal and access the shell.
3.	Mount the root partition:

```
sudo mkdir /mnt/root
sudo mount /dev/sda3 /mnt/root
sudo mount /dev/sda1 /mnt/root/boot/efi
sudo chroot /mnt/root
```
4.	**Update the system:**

```
apt update
apt full-upgrade
```

5.	Install official NVIDIA drivers (recommended):

```
sudo apt install nvidia-driver-570
```

6.	(Optional but recommended):

```
sudo apt install system76-driver-nvidia
```

7.	**Check EFI file:**

- op!_OS bootloader should automatically update:

```
/mnt/root/boot/efi/loader/entries/Pop_OS-current.conf
```
 - Ensure the correct line is present:

```
quiet splash nvidia-drm.modeset=1
```

8.	**Reboot.**

⸻

#### ✅ 5️⃣ FINAL NVIDIA DRIVER VERIFICATION

After reboot:
- Confirm GPU management:

```
nvidia-smi
```

- Check active OpenGL driver:

```
sudo apt install mesa-utils
glxinfo | grep "OpenGL renderer"
```

- Check loaded kernel modules:

```
lsmod | grep nvidia
```

- Diagnose possible errors:

```
dmesg | grep nvidia
```

⸻

#### 🏁 6️⃣ FINAL SYSTEM STATUS

If everything is correctly configured:
- he system will boot into the graphical interface using the NVIDIA driver natively.
- Operating system selection (Pop!_OS or Windows) is handled via BIOS boot menu.

⸻

> 📌 FINAL NOTE

> You can re-enable Secure Boot after installation only if:
> - You use the MOK (Machine Owner Key) module.
> - You manually sign the NVIDIA kernel module.

> Otherwise, it’s best to leave Secure Boot disabled to avoid complications.
