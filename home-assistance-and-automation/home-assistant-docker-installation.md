# 🏠 Home Assistant Container Installation Guide

Welcome to the installation guide for Home Assistant using Docker. This guide will walk you through the steps to set up Home Assistant in a Docker container, allowing you to control and automate your smart home devices.

## Prerequisites 🧰
Before we begin, ensure that you have the following:
- A machine with Ubuntu Server 22.04 installed and connected to the network.
- Docker installation.

## Step 1: Pull and Run Home Assistant Container 🏃‍♂️
To install Home Assistant using Docker, follow these steps:

1. Open a terminal or SSH into your machine.

2. Pull the latest stable Home Assistant Docker image and run the container with the following command:

   ```bash
   docker run -d \
   --name homeassistant \
   --privileged \
   --restart=unless-stopped \
   -e TZ=MY_TIME_ZONE \
   -v /PATH_TO_YOUR_CONFIG:/config \
   -v /run/dbus:/run/dbus:ro \
   --network=host \
   ghcr.io/home-assistant/home-assistant:stable
   ```

3. Replace MY_TIME_ZONE with your desired timezone (e.g., Europe/Rome).
4. Replace /PATH_TO_YOUR_CONFIG with the folder path where you want to store your Home Assistant configuration.
5. Remember to install bluetooth integration on your machine (https://github.com/bus1/dbus-broker/wiki).
6. Wait for a few moments for the container to start. You can check the container logs using the following command:

```bash
docker logs -f homeassistant
```

7. Once you see a log message indicating that Home Assistant has started successfully, you can proceed to the next step.

## Step 2: Access Home Assistant Web Interface 🌐

To access the Home Assistant web interface, follow these steps:

1. Open a web browser on any device connected to the same network as your machine.
2. Enter the following URL in the address bar:

```arduino
http://<YOUR_IP_ADDRESS>:8123
```

3. Replace <YOUR_IP_ADDRESS> with the IP address of your machine.
4. Wait for the Home Assistant interface to load. This might take a few moments during the initial setup.
5. Create an account and set up your Home Assistant instance as per your requirements.

## Congratulations! 🎉
You have successfully installed and accessed Home Assistant on your machine using Docker.

For more information, configuration options, and troubleshooting tips, please refer to the official Home Assistant documentation.
