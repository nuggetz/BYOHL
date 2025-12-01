# 🌐 WireGuard VPN Setup with ProtonVPN using systemd-networkd and wpasupplicant

What this guide does – It shows how to install and run a WireGuard client directly on the endpoint (your Linux machine) instead of relying on a router‑based VPN. By configuring the VPN on the host itself you keep full control of the tunnel, avoid having to expose the VPN to the whole LAN, and can easily switch or disable the connection per‑device. The steps below walk you through generating the ProtonVPN configuration, placing it where systemd‑networkd can use it, and managing the service with systemd‑resolved and systemd‑networkd. ⚠️ **Note**: Using Network Manager or similar tools may interfere with this setup, but it's still worth trying due to its ease of setup and ability to revert if necessary.

## 📝 Prerequisites

- 🖥️ Linux system with `systemd-networkd`, `wpasupplicant`, and `systemd-resolved` for network management.
- 🛡️ ProtonVPN account.

## 1️⃣ Step 1: Generate WireGuard Config on ProtonVPN

Go to the ProtonVPN dashboard and generate a WireGuard configuration file. You'll need to pick a specific server during this process. ProtonVPN currently does not offer an automatic server selection option.

## 2️⃣ Step 2: Download and Save the Config

1. Download the generated configuration file.
2. Move the file to `/etc/wireguard/` with a **deterministic** name, such as `wg<country><servername>.conf`. For example, you might use `wgusny5.conf` for a New York server in the US.

```bash
sudo mv ~/Downloads/wgusny5.conf /etc/wireguard/
```

## 3️⃣ Step 3: Setup DNS Resolver (if needed) 🌐

If you're using `systemd-resolved` and don't have `resolvconf` installed, create a symlink to make `resolvectl` act as `resolvconf`.

```bash
sudo ln -s /usr/bin/resolvectl /usr/local/bin/resolvconf
```

If `resolvconf` is already installed, skip this step. If you're using another DNS resolver, check its documentation for how to emulate `resolvconf`.

## 4️⃣ Step 4: Enable and Start WireGuard at Boot 🛠️

Enable WireGuard to start automatically at boot:

```bash
sudo systemctl enable --now wg-quick@<configfile>
```

Replace `<configfile>` with the name you chose in step 2, for example, `wgusny5.conf`. This command will both enable and start the VPN connection.

## 5️⃣ Step 5: Switching Between VPN Servers 🌍

To switch between different VPN servers, repeat steps 1 and 2 for the new server. For example, if you have both `wgusny5.conf` and `wguklondon2.conf`, you can switch between servers with:

```bash
sudo systemctl stop wg-quick@wgusny5
sudo systemctl start wg-quick@wguklondon2
```

By default, on boot, the VPN will connect to the server you enabled with `systemctl`. To change the default server at boot, disable the current server and enable the new one:

```bash
sudo systemctl disable wg-quick@wgusny5
sudo systemctl enable wg-quick@wguklondon2
```

## 6️⃣ Step 6: Disable VPN Autostart 🚫

If you want to stop the VPN from starting automatically at boot, you can disable it with:

```bash
sudo systemctl disable wg-quick@<configfile>
```

To stop the VPN connection immediately and prevent it from starting at boot:

```bash
sudo systemctl disable --now wg-quick@<configfile>
```

## 🔚 Final Notes

This method may require terminal usage 🖥️, but it keeps things simple and reliable. In my experience, this approach "Just Works" with fewer issues compared to graphical tools like Network Manager.

If you're switching from OpenVPN, the process is similar—simply swap `openvpn@` for `wg-quick@` in the commands. 

---

🙏 **Credit**: Huge thanks to [chiraagnataraj](https://www.reddit.com/user/chiraagnataraj) on Reddit for the original guide and inspiration! 🎉
