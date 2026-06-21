---
title: "Docker Compose"
description: "Define and run the Certer service configuration using Docker Compose."
icon: "layers"
weight: 220
---

Docker Compose lets you define the Certer daemon stack declaratively in a `docker-compose.yml` file, making local testing and server configuration simple.

---

## 1. Directory Structure

Create a new directory for your stack and arrange the files as follows:

```text
certer-stack/
├── .env
├── config.json
└── docker-compose.yml
```

---

## 2. Docker Compose File (`docker-compose.yml`)

Define the Certer service, linking local directories and configuring restart policies:

```yaml
version: '3.8'

services:
  certer:
    image: m0space/certer:latest
    container_name: certer-daemon
    ports:
      - "8080:8080"
    volumes:
      - ./certs:/app/certs
      - ./config.json:/app/config.json
    environment:
      - CLOUDFLARE_EMAIL=${CLOUDFLARE_EMAIL}
      - CLOUDFLARE_API_KEY=${CLOUDFLARE_API_KEY}
      - LOG_LEVEL=${LOG_LEVEL:-info}
      - LOG_STYLE=${LOG_STYLE:-json}
    restart: unless-stopped
```

---

## 3. Environment Variables (`.env`)

Configure your DNS API variables locally in a `.env` file. These values will be automatically injected into your compose configuration when executing:

```ini
# Cloudflare ACME credentials
CLOUDFLARE_EMAIL=you@example.com
CLOUDFLARE_API_KEY=your_private_cloudflare_dns_api_token

# Daemon configuration defaults
LOG_LEVEL=info
LOG_STYLE=json
```

---

## 4. Run the Stack

1. Make sure your [`config.json`](../docker/#step-2-create-a-minimal-configuration-configjson) file is created in the same directory.
2. Start the daemon using:
   ```bash
   docker compose up -d
   ```
3. Watch logs in real-time:
   ```bash
   docker compose logs -f
   ```
4. To stop the daemon, run:
   ```bash
   docker compose down
   ```
