# physrisk-deploy
Deployment of physrisk API and UI using Docker Compose, with Caddy handling HTTPS automatically via Let's Encrypt.

**Containers:**
- `quay.io/os-climate/physrisk-api` — FastAPI backend (internal port 8081)
- `quay.io/os-climate/physrisk-ui` — React frontend (internal port 80)
- `caddy:2-alpine` — reverse proxy, public ports 80/443

Caddy routes `/api/*` to the API container and everything else to the UI container.

## Deploy to Lightsail

### 1. Create and configure the Lightsail instance

- Create a Lightsail instance running **Ubuntu 24.04 LTS** (2 GB RAM minimum).
- Attach a **static IP** to the instance.
- In the instance **Networking** tab, open inbound ports **80 (TCP)** and **443 (TCP + UDP)**.

### 2. Point DNS to the instance

Add an **A record** at your DNS provider pointing `physrisk.com` (and `www.physrisk.com`) to the Lightsail static IP `34.228.204.143`. Propagation can take a few minutes to an hour.

### 3. Install Docker on the instance

SSH into the instance, then:

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
newgrp docker
```

### 4. Copy files to the instance

From your local machine:

```bash
scp -i your-key.pem docker-compose.yml Caddyfile .env.example ubuntu@34.228.204.143:~/physrisk/
```

Or clone this repo directly on the instance:

```bash
git clone https://github.com/YOUR_ORG/physrisk-deploy.git ~/physrisk
```

### 5. Configure the Caddyfile

The Caddyfile is already configured for `physrisk.com`. If your domain changes, update the three occurrences in `Caddyfile` and the admin email address.

### 6. Create the `.env` file

```bash
cp .env.example .env
nano .env   # add any environment variables required by the API container
```

If the API needs no environment variables, leave the file empty — it must still exist.

### 7. Start the stack

```bash
docker compose up -d
```

Caddy will obtain a Let's Encrypt certificate on first start. After 30–60 seconds visit `https://physrisk.com` to confirm everything is running.

### Useful commands

```bash
docker compose logs -f           # stream all logs
docker compose logs -f caddy     # Caddy/HTTPS logs only
docker compose logs -f api       # API logs only
docker compose pull              # pull latest images
docker compose up -d             # restart with updated images
docker compose down              # stop and remove containers
```
