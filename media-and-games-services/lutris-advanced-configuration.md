# 🎮 Advanced Guide to Lutris on Pop!_OS (Flatpak - NVIDIA)

This guide explains how to turn **Lutris** into a centralized manager for Windows games, manually installing launchers like **Battle.net**, **Epic Games**, **EA Play**, and **Ubisoft Connect**.
We’ll work using the **Flatpak** version of Lutris, with Wine GE, GameMode, and MangoHud integrated to ensure smooth gaming performance.

⸻

#### 📌 Introduction: What is Lutris

Lutris is an open-source game manager designed for Linux. It doesn’t integrate official launchers (like Steam does with Epic), but it allows installing and managing them as standalone games via:
- **Runners:** execution environments (e.g., Wine GE).
- **Installation scripts:** automate launcher setup.
- **Environment variables:** optimize game execution.

⸻

#### ⚙️ 1. Installing Wine GE (GloriousEggroll)

Even if you use Proton GE in Steam, Lutris requires **Wine GE** (a modified Wine version).
Manual installation of Wine GE:
	1.	Download Wine GE:
https://github.com/GloriousEggroll/wine-ge-custom/releases
	2.	Create the directory:
 
```
mkdir -p ~/.var/app/net.lutris.Lutris/data/lutris/runners/wine/
```

	3.	Extract Wine GE:

```
tar -xf ~/Downloads/wine-ge-*-x86_64.tar.xz -C ~/.var/app/net.lutris.Lutris/data/lutris/runners/wine/
```

Lutris will automatically detect Wine GE as an available runner.

⸻

#### ⚙️ 2. Global Lutris Configuration

Open Lutris > Preferences:
- **Default runner:** Wine-GE (the one you just installed).
- **GameMode:**
- In each game > System Options:
- Add environment variable:
- **Key:** LD_PRELOAD
- **Value:** /app/lib/libgamemodeauto.so.0
- **MangoHud:**
-       In each game > System Options:
-       **Enable MangoHud:** ✓ (if you want the FPS overlay).

⸻

#### 🎮 3. Installing Launchers (Manual Method)

General procedure:
	1.	Search for your desired launcher at lutris.net/games/.
	2.	Open Lutris.
	3.	Click + > **Install from a local install script URL**, or directly:
	-      Lutris > Browser Games > Search launcher > Click Install.

**Specific launchers:**

**🛡️ Battle.net**
	1.	Search “Battle.net” at:
https://lutris.net/games/battlenet/
	2.	Select the official (updated) installer.
	3.	Follow the process: it will install Wine GE, DXVK, and the launcher.

**🎁 Epic Games Store**
	1.	Search “Epic Games Store” at:
https://lutris.net/games/epic-games-store/
	2.	Install via script.
	3.	After login, manage games like on Windows.

**🎮 EA Play (via Origin)**
	1.	Search “Origin” at:
https://lutris.net/games/origin/
	2.	Install. (Note: EA App is not yet stable on Wine.)

**🎮 Ubisoft Connect**
	1.	Search “Ubisoft Connect” at:
https://lutris.net/games/ubisoft-connect/
	2.	Install via script.

⸻

#### ⚙️ 4. Game Management

Each launcher will appear as a “game” inside Lutris. From there, you can:
- Launch the launcher itself.
- Install and manage Windows games like on Windows.
- Treat each installed game as a separate app.

**Tip:**
- After installing a game via launcher, you can add it as a dedicated entry in Lutris:
-       Right-click on the launcher > **Create Shortcut** for installed games.

⸻

#### 💡 5. Tips and Best Practices
- Use **Wine GE** as the default runner.
- Keep **GameMode** active via environment variables.
- Enable **MangoHud** for FPS/debug overlays.
- Use official **Lutris scripts** to automate setups.
- Periodically update Wine GE by downloading new versions.

⸻

#### 📋 Conclusion

Lutris now allows you to:
- Run Windows launchers (Battle.net, Epic, Ubisoft Connect, EA Play).
- Install and play games just like in Windows.
- Enjoy optimized performance thanks to Wine GE, GameMode, and MangoHud.

 ⸻
