# 📦 Homarr – Docker‑Compose Installation Guide

Below is a complete, step‑by‑step guide to get Homarr up and running with Docker‑Compose, from prerequisites to first‑run verification and optional TLS/reverse‑proxy configuration.

---

## 1️⃣ Prerequisites

| Requirement |	How to satisfy |
|---------------------------------|------------------------------------------------------------------------------------------|
| Docker Engine (≥ 20.10)	| `curl -fsSL https://get.docker.com` |
| Docker‑Compose (v2 plugin) | `sudo apt-get install docker-compose-plugin` (or `pip install docker-compos`e for other distros) |
| A directory for persistent data |	e.g. `/opt/homarr` (any path you like) |
| (Optional) Domain name	| Needed only if you want HTTPS via a reverse proxy. |
| (Optional) Reverse‑proxy	| Caddy, Nginx, Traefik, … – we’ll show a minimal Caddy example. |

> Tip: Run all commands as a user that belongs to the `docker` group, or prefix with `sudo`.

---

## 2️⃣ Directory layout

```bash
/home/your‑user/
└─ homarr/
   ├─ docker-compose.yml      ← will be created in the next step
   └─ appdata/               ← persistent Homarr data (auto‑created)
Create the base folder and give it proper permissions:

mkdir -p /home/your-user/homarr/appdata
chmod -R 755 /home/your-user/homarr
```

---

## 3️⃣ Generate a SECRET_ENCRYPTION_KEY

Homarr encrypts sensitive data (API tokens, passwords) with this key.
Generate a 64‑character hex string (or any strong random string) once and store it safely – you’ll need it again if you ever recreate the container.

**Example using openssl** (hex, 32 bytes = 64 hex chars)
openssl rand -hex 32
Copy the output; we’ll paste it into the docker-compose.yml file.

---

## 4️⃣ docker-compose.yml

Create (or edit) /home/your-user/homarr/docker-compose.yml with the following content. Replace <YOUR_SECRET_KEY> with the value you generated above and adjust the host‑side paths if you placed the folder elsewhere.

```bash
version: "3.9"

services:
  homarr:
    container_name: homarr
    image: ghcr.io/homarr-labs/homarr:latest
    restart: unless-stopped

    # Mount the Docker socket only if you want Homarr to display
    # running containers / allow “Docker integration”.
    # Remove the line if you don’t need that feature.
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./appdata:/appdata            # <-- persistent Homarr data

    environment:
      - SECRET_ENCRYPTION_KEY=<YOUR_SECRET_KEY>

    # Expose the UI on port 7575 (host:container)
    ports:
      - "7575:7575"
```

Explanation of the fields

Field	Why it matters
container_name	Easy to reference with docker exec -it homarr …
restart: unless-stopped	Guarantees Homarr survives reboots but won’t restart if you manually stop it.
volumes	appdata holds your dashboards, plugins, and encrypted secrets. The Docker socket is optional – it enables the “Docker integration” widget.
environment	SECRET_ENCRYPTION_KEY protects stored credentials.
ports	Maps the internal port 7575 to the same port on the host. You can change the host side (e.g., 8080:7575) if you need a different public port.

---

## 5️⃣ Bring the stack up

```bash
cd /home/your-user/homarr
docker compose up -d
Docker will pull the image, create the container, and start it in detached mode.
```

Verify it’s running

```bash
docker ps --filter name=homarr
```

You should see something like:

```bash
CONTAINER ID   IMAGE                                 COMMAND   CREATED          STATUS          PORTS                    NAMES
a1b2c3d4e5f6   ghcr.io/homarr-labs/homarr:latest    "/init"   10 seconds ago   Up 9 seconds    0.0.0.0:7575->7575/tcp   homarr
Open a browser and navigate to http://<your‑host‑IP>:7575. The first‑time setup wizard will ask you to create an admin account.
```

---

## 6️⃣ (Optional) Secure the UI with HTTPS

Running Homarr exposed on plain HTTP is fine on a trusted LAN, but for remote access you’ll want TLS. The simplest way is to put a lightweight reverse‑proxy in front of Homarr.

#### 6.1 Minimal Caddy example
Create a caddy/Caddyfile next to your docker-compose.yml:

```bash
your.domain.com {
    reverse_proxy homarr:7575
    tls your@email.com          # obtains a cert from Let's Encrypt
}
```

Add the Caddy service to the same docker-compose.yml:

```bash
  caddy:
    image: caddy:latest
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./caddy/Caddyfile:/etc/caddy/Caddyfile
      - caddy_data:/data
      - caddy_config:/config
    depends_on:
      - homarr

volumes:
  caddy_data:
  caddy_config:
```

Now run:

```bash
docker compose up -d
```

Visit https://your.domain.com – Caddy will automatically fetch a certificate and forward traffic to Homarr.

Alternative reverse proxies – Nginx, Traefik, or Apache work equally well; just replace the Caddy block with the appropriate configuration.

---

## 7️⃣ Updating Homarr

When a new version is released, simply pull the latest image and restart:

```bash
docker compose pull homarr   # fetch latest tag
docker compose up -d --no-deps --force-recreate homarr
```

Your appdata volume ensures all dashboards and settings survive the upgrade.

---

## 8️⃣ Common troubleshooting

Symptom	Quick fix
Container exits immediately	Run docker logs homarr to see the error. Most often it’s a missing SECRET_ENCRYPTION_KEY or permission issue on ./appdata.
Cannot reach UI on port 7575	Verify the host firewall (ufw status or iptables -L). Allow the port or use the reverse‑proxy method.
Docker integration widget shows “No containers”	Ensure the Docker socket is mounted (/var/run/docker.sock:/var/run/docker.sock) and that the user running the container has permission to read the socket (usually root inside the container).
TLS certificate fails	Check that port 80 is reachable from the internet (Let’s Encrypt needs it for the HTTP‑01 challenge). If you’re behind a NAT, forward ports 80 and 443 to the host.

---

## 9️⃣ Clean removal

If you ever want to delete Homarr completely:

```bash
docker compose down -v   # stops containers and removes the anonymous volumes
rm -rf /home/your-user/homarr   # deletes persisted data
```

---

## 🎉 You’re ready!
Your Homarr dashboard is now running in a reproducible Docker‑Compose stack, optionally secured with HTTPS. Feel free to customize the UI, add widgets, or integrate it with the Docker socket for live container monitoring. Happy dashboarding! 🚀
