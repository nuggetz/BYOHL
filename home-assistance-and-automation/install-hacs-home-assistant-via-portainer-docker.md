# Install HACS on Home Assistant (Docker) via Portainer Terminal

👋 Hello! Welcome to this guide on installing **HACS** (the community store) on **Home Assistant** installed via **Docker** using the **Portainer** terminal. 🐳

## Prerequisites

Before we begin, make sure you have the following:

- A running instance of **Home Assistant** installed on **Docker**
- Access to **Portainer**, the Docker container management tool

## Step-by-Step Installation Guide

1. Open your **web browser** and navigate to the **Portainer interface** (http://YOUR_HOMESERVER_IP:9000)
2. Click on "**Local**" to access your local Docker environment
3. From the left-hand menu, select "**Containers**" to view the list of running containers
4. Locate and select the container for Home Assistant
5. In the "**Quick Actions**" section, click on "**Exec Console**"
6. Click on "**Connect**" to open the terminal for the Home Assistant container
7. In the terminal, enter the following command and press Enter:

   ```shell
   wget -O - https://get.hacs.xyz | bash -
   ```
   *This command will download and execute the HACS installation script.*

8. Wait for the installation to complete. You will see output indicating the progress of the installation process
9. Once the installation is finished, return to the "**Containers**" page in Portainer
10. Locate and select the container for Home Assistant
11. Click on "**Restart**" to restart the Home Assistant container
12. Wait for Home Assistant to restart, and you're done! 🎉

    Now you have successfully installed **HACS** on **Home Assistant** using Portainer terminal within Docker.

## Additional Tips

- To access HACS, open your web browser and navigate to the Home Assistant interface. You should now go to "**Settings**", "**Devices**" and search for HACS in the integrations drop down menu.
- After that you should see a new section dedicated to managing and installing community integrations and plugins on the right.
- If you encounter any issues during the installation, please refer to the HACS documentation and the Home Assistant community for support.

That's it! You're now ready to explore and enhance your Home Assistant experience with the power of HACS. Enjoy! 😊
