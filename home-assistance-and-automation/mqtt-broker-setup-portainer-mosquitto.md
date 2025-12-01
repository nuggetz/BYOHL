# 🌐 MQTT Broker Setup Guide via Portainer

This guide provides detailed instructions on how to set up an MQTT broker using the Eclipse Mosquitto image in Portainer, complete with volume mapping and initial configuration.

## Prerequisites

- Access to the Portainer web interface.
- Basic familiarity with MQTT and Docker.

## 📦 Step 1: Creating volumes for container persistance

1. Launch your web browser and navigate to your Portainer dashboard (`http://<your-server-ip>:9000`).
2. Go to the "Volumes" section in Portainer and add three volumes:
       - `mqtt_data` (leave other settings as default)
       - `mqtt_config` (leave other settings as default)
       - `mqtt_log` (leave other settings as default)

## 🚀 Step 2: Add a Container

- Navigate to the "Containers" section and click on "Add container".
- Name the container `mqtt`.
- Set the image to be `eclipse-mosquitto`.

## 🔄 Step 3: Container Settings

1. Enable the option to "Always pull the image" to ensure you're using the latest version.
2. Set the console to interactive.
3. In the volumes tab, map the previously created volumes to the respective paths:
     - `/mosquitto/config` to `mqtt_config`
     - `/mosquitto/data` to `mqtt_data`
     - `/mosquitto/log` to `mqtt_log`
4. Set the restart policy to "unless stopped".
5. Click "Deploy the container" to start your MQTT broker.

## 🔌 Step 4: Configure MQTT Broker

1. Once the container is ready, return to the "Containers" section, find your MQTT container, and under quick actions, select "Exec".
2. Connect to access the MQTT broker container console.
3. Navigate to the configuration directory:

  ```bash
  cd mosquitto/config
  ```

4. Make a backup of the existing config:

  ```bash
  cp mosquitto.conf mosquitto.conf.orig
  ```

5. Open the configuration file for editing:

  ```bash
  vi mosquitto.conf
  ```

6. In `vi`, type `:%d` to delete every line, then press `i` to enter insert mode.
7. Paste the following configuration:

  ```bash
  persistence true
  persistence_location /mosquitto/data/
  user mosquitto
  listener 1883
  allow_anonymous true
  log_dest file /mosquitto/log/mosquitto.log
  log_dest stdout
  ```

8. Press `esc`and type `:wq` to save and quit.
9. Exit the console and restart the container to apply the changes.

## ✅ Congratulations!

Your MQTT broker is now set up and configured. It’s ready to handle messages!
