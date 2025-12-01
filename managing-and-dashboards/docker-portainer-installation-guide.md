# :whale: Docker and Portainer Installation Guide

This guide provides step-by-step instructions for installing Docker and Portainer on your server.

## Prerequisites

- A server or VM running Linux Server 22+.
- Access to the command-line interface (CLI) on your server.

## Step 1: Update System Packages

Before installing Docker and Portainer, it's recommended to update the system packages to ensure you have the latest versions. Open the terminal on your server and run the following command:

```shell
sudo apt update && sudo apt upgrade -y
```

## Step 2: Install Docker

Docker allows you to run applications in containers, providing an efficient and isolated environment. To install Docker, run the following commands:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
```

After executing these commands, you need to log out and then log back in to apply the group permission changes.

To verify that Docker is installed correctly, run the following command:

```shell
docker version
```

You should see the Docker version information displayed.

To test Docker and ensure you have the correct permissions, you can create a test container:

```shell
docker run hello-world
```

## Step 3: Install Portainer

First, create the volume that Portainer Server will use to store its database:

```plaintext
docker volume create portainer_data
```

Then, download and install the Portainer Server container:

```plaintext
docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:lts
```
By default, Portainer generates and uses a self-signed SSL certificate to secure port 9443. Alternatively you can provide your own SSL certificate during installation or via the Portainer UI after installation is complete.

If you require HTTP port 9000 open for legacy reasons, add the following to your docker run command:

```plaintext
-p 9000:9000
```

## Step 4: Access Portainer

To access Portainer, open a web browser on a device connected to the same network as your server. Enter the following address in the browser's URL bar:

```plaintext
https://<your-server-ip>:9443
```

Replace `<your-server-ip>` with the IP address of your server.

You will be prompted to create an initial admin user and password. Follow the on-screen instructions to complete the setup.

Once logged in, you can start managing your Docker environment using the Portainer web interface.

## Step 5: Setting Up Portainer Agent (optional)

To manage multiple Docker environments from your central Portainer instance, set up Portainer Agents on the other machines.

1. **Install Docker on Other Machines**: Ensure Docker is installed on all machines you wish to manage with Portainer.

2. **Install Portainer Agent**: On each machine, install the Portainer Agent by executing:

   ```shell
   docker run -d -p 9001:9001 --name portainer_agent --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v /var/lib/docker/volumes:/var/lib/docker/volumes portainer/agent
   ```

3. **Connect to Main Portainer Instance**: In your main Portainer UI, navigate to the 'Environments' (or 'Endpoints' in older versions) section, and click 'Add environment'. Enter the IP address or hostname of the machine running the Portainer Agent, specifying port `9001`.

5. **Verify the Connection**: After adding the environment, it should appear in your Portainer dashboard, allowing you to manage the Docker environment on the remote machine from your main Portainer interface.

## Step 6: Add Docker to APT Repository and Install Compose Plug in (optional but very recomended)

1. Set up Docker's `apt` repository.

```bash
   # Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

2. Update the package index, and install the latest version of `Docker Compose`:

```bash
sudo apt-get update
sudo apt-get install docker-compose-plugin
```

3. Verify that Docker Compose is installed correctly by checking the version.

```bash
docker compose version
```

# Updating on Docker Standalone
Always match the agent version to the Portainer Server version. In other words, when you're installing or updating to Portainer 2.27.6 make sure all of the agents are also on version 2.27.6.

If you are updating from the 1.x version of Portainer, you must first update to 2.0.0 before updating to the newest version or you will run into issues.

Before beginning any update, we highly recommend taking a backup of your current Portainer configuration.

Updating your Portainer Server
Starting from Portainer CE 2.9 and BE 2.10, HTTPS is enabled by default on port 9443. These instructions will configure Portainer to use 9443 for HTTPS and do not expose 9000 for HTTP. If you need to retain HTTP access, you can add:

-p 9000:9000

to your command.

You can also choose to completely disable HTTP after the update. Before you make Portainer HTTPS only, make sure you have all your Agents and Edge Agents already communicating with Portainer using HTTPS.

This article assumes that you used our recommended deployment scripts.

To update to the latest version of Portainer Server, use the following commands to stop then remove the old version. Your other applications/containers will not be removed.

```bash
docker stop portainer
```

```bash
docker rm portainer
```

Now that you have stopped and removed the old version of Portainer, you must ensure that you have the most up to date version of the image locally. You can do this with a docker pull command:

```bash
docker pull portainer/portainer-ce:lts
```

Finally, deploy the updated version of Portainer:

```bash
docker run -d -p 8000:8000 -p 9443:9443 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:lts
```

These docker run commands include opening port 8000 which is used for Edge Agent communication as included in our installation instructions. If you do not need this port open, you can remove it from the command.

To provide your own SSL certs you may use --sslcert and --sslkey flags as below to provide the certificate and key files. The certificate file needs to be the full chain and in PEM format. For example, for Business Edition:

```bash
docker run -d -p 8000:8000 -p 9443:9443 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ee:lts --sslcert /path/to/cert/portainer.crt --sslkey /path/to/cert/portainer.key
```

The newest version of Portainer will now be deployed on your system, using the persistent data from the previous version, and will also upgrade the Portainer database to the new version.

When the deployment is finished, go to https://your-server-address:9443 or http://your-server-address:9000 and log in. You should notice that the update notification has disappeared and the version number has been updated.

# Agent-only update
To update to the latest version of Portainer Agent, use the following commands to stop then remove the old version. Your other applications/containers will not be removed.

```bash
docker stop portainer_agent
```

```bash
docker rm portainer_agent
```

Next, pull the updated version of the image:

```bash
docker pull portainer/agent:lts
```

Finally, start the agent with the updated image:

```bash
docker run -d -p 9001:9001 --name portainer_agent --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v /var/lib/docker/volumes:/var/lib/docker/volumes portainer/agent:lts
```

If you have set a custom AGENT_SECRET on your Portainer Server instance (by specifying an AGENT_SECRET environment variable when starting the Portainer Server container) you must remember to explicitly provide the same secret to your Agent in the same way (as an environment variable) when
updating your Agent:

```Bash
-e AGENT_SECRET=yoursecret
```
