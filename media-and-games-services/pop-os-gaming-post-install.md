# 🚀 Pop!_OS Gaming Post-Install Guide (NVIDIA – Flatpak Focus)

This guide covers the essential steps to turn Pop!_OS into a stable and modern gaming machine, leveraging NVIDIA hardware and Linux-native technologies. It is designed for a clean installation, using Steam via Flatpak, Proton GE, Lutris, MangoHud, and GameMode.

> ⚠️ Advanced configurations (e.g., customized MangoHud) will be covered in a separate guide.

⸻

#### 🎮 1. Update the System

```bash
sudo apt update && sudo apt upgrade -y
flatpak update
```

⸻

#### 🎮 2. Install Steam (Flatpak)

```bash
flatpak install flathub com.valvesoftware.Steam
```

**To launch Steam:**

```bash
flatpak run com.valvesoftware.Steam
```

⸻

#### 🍷 3. Proton GE – GloriousEggroll (Version 10-9)


**Download Proton GE:**

```bash
https://github.com/GloriousEggroll/proton-ge-custom/releases
```

**Example file:**

```Bash
GE-Proton10-9.tar.gz
```

**Extract Proton GE:**

```Bash
mkdir -p ~/.var/app/com.valvesoftware.Steam/data/Steam/compatibilitytools.d/
tar -xf ~/Downloads/GE-Proton10-9.tar.gz -C ~/.var/app/com.valvesoftware.Steam/data/Steam/compatibilitytools.d/
```

**Enable Proton GE in Steam:**

- Steam > Settings > Steam Play
- Enable for all titles
- Select GE-Proton10-9 as the version

⸻

#### 🍻 4. Install Lutris

```Bash
flatpak install flathub net.lutris.Lutris
```

**To launch Lutris:**

```Bash
flatpak run net.lutris.Lutris
```

⸻

#### 📊 5. Install MangoHud

**For Steam Flatpak:**

```Bash
flatpak install flathub org.freedesktop.Platform.VulkanLayer.MangoHud
```

**For native games:**

```Bash
sudo apt install mangohud
```

⸻

#### ⚙️ 6. Install and Configure GameMode

```Bash
sudo apt install gamemode libgamemode0 libgamemodeauto0
```

**(Optional) Start and enable the GameMode service:**

```bash
systemctl --user start gamemoded
systemctl --user enable gamemoded
```

**🎮 Using GameMode in Steam (Flatpak)**

For each game:
- Go to **Properties > Launch Options**
- Add the following line:

```bash
gamemoderun %command%
```

This makes Steam invoke GameMode for that specific game.

⸻

#### 🍷 Using GameMode in Lutris (Flatpak)

For each game:

- Right-click the **game > Configure**
- Go to **System Options**
- In **Environment Variables**, add:
 
  - **Key:** LD_PRELOAD
  - **Value:** /app/lib/libgamemodeauto.so.0

This will preload GameMode in games launched through Lutris Flatpak.

⸻

#### 📋 Conclusion

Your Pop!_OS is now ready for gaming:

- ✅ System updated
- ✅ Official NVIDIA drivers installed
- ✅ Steam (Flatpak) configured
- ✅ Proton GE operational
- ✅ Lutris installed
- ✅ MangoHud working
- ✅ GameMode active in games

⸻

> **🛠️ Advanced configurations will be addressed in a dedicated guide.**

⸻
