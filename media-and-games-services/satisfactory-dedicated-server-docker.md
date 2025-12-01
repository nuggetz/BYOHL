# 🚀 Satisfactory Dedicated Server Installation Guide

Welcome to this guide! Follow these steps to install and run a dedicated server for Satisfactory on Linux. This guide assumes you're using a **Ubuntu** system, but it should work similarly for other Linux distributions. 🌍

---

## Prerequisites 📋

Before starting, ensure that your system has `SteamCMD` installed. This can be done by enabling the **multiverse** repository and i386 architecture.

1. **Enable Multiverse Repository and Install SteamCMD**:
    ```bash
    sudo add-apt-repository multiverse; sudo dpkg --add-architecture i386; sudo apt update
    sudo apt install steamcmd
    ```

2. **Non-root User** 👤:
    To avoid permission issues, use a non-root user for the server installation and management.
   
---

## Step 1: Install the Satisfactory Dedicated Server

1. **Use SteamCMD to Install Satisfactory Dedicated Server**:
    ```bash
    steamcmd +force_install_dir ~/SatisfactoryDedicatedServer +login anonymous +app_update 1690800 -beta public validate +quit
    ```

2. **Navigate to the installation folder** and launch the server:
    ```bash
    cd ~/SatisfactoryDedicatedServer
    ./FactoryServer.sh
    ```

---

## Step 2: Setup Systemd Service for Automatic Management 🔄

We'll create a Systemd service that:
- **Automatically checks for updates** when the server starts.
- **Starts the server on reboot** of the host system.

1. **Create the Satisfactory Systemd Unit File** 📝:

    The systemd unit file will be located at `/etc/systemd/system/satisfactory.service`. You can create it using a text editor (e.g., `nano`):

    ```bash
    sudo nano /etc/systemd/system/satisfactory.service
    ```

    Paste the following into the file (remember to modify `your_user` and paths as needed):

    ```ini
    [Unit]
    Description=Satisfactory dedicated server
    Wants=network-online.target
    After=syslog.target network.target nss-lookup.target network-online.target

    [Service]
    Environment="LD_LIBRARY_PATH=./linux64"
    ExecStartPre=/usr/games/steamcmd +force_install_dir /home/your_user/SatisfactoryDedicatedServer +login anonymous +app_update 1690800 validate +quit
    ExecStart=/home/your_user/SatisfactoryDedicatedServer/FactoryServer.sh -ServerQueryPort=15777 -BeaconPort=15000 -Port=7777 -log -unattended -multihome=0.0.0.0
    User=your_user
    Group=your_user
    StandardOutput=journal
    Restart=on-failure
    WorkingDirectory=/home/your_user/SatisfactoryDedicatedServer

    [Install]
    WantedBy=multi-user.target
    ```

    Make sure to replace `your_user` with the username you created earlier and adjust any server parameters to suit your needs. 

2. **Log Files to Disk** (Optional) 📄:
   
   If you want to log server output to files instead of journaling, add these lines inside the `[Service]` section:

   ```ini
   StandardOutput=append:/var/log/satisfactory.log
   StandardError=append:/var/log/satisfactory.err
   ```

---

## Step 3: Finalize and Start the Server 🚦

1. **Reload Systemd Daemon** to load the new service:
    ```bash
    sudo systemctl daemon-reload
    ```

2. **Enable the service to start on boot**:
    ```bash
    sudo systemctl enable satisfactory
    ```

3. **Start the Satisfactory service**:
    ```bash
    sudo systemctl start satisfactory
    ```

4. **Check the service status**:
    ```bash
    sudo systemctl status satisfactory
    ```

    You should see output similar to this if the server is running correctly:

    ```text
    ● satisfactory.service - Satisfactory dedicated server
        Loaded: loaded (/etc/systemd/system/satisfactory.service; enabled; vendor preset: enabled)
        Active: active (running) since [date]; [time]
        Main PID: XXXX (FactoryServer.sh)
        ...
    ```

---

## 🚧 Troubleshooting

- **Service Fails to Start**:
    - Check for missing dependencies or permissions.
    - Make sure all paths and usernames are correctly configured.
  
- **Logs and Errors**:
    - If you enabled file logging, check `/var/log/satisfactory.log` and `/var/log/satisfactory.err`.

---

🎉 **Congratulations! Your Satisfactory dedicated server should now be up and running!** Happy building, Pioneer!

--- 
