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
    image: ghcr.io/menschomat/certer:latest
    container_name: certer-daemon
    ports:
      - "8080:8080"
    volumes:
      - ./certs:/certs
      - ./config.json:/config.json
    environment:
      - DNS_PROVIDER=${DNS_PROVIDER:-cloudflare}
      - CF_DNS_API_TOKEN=${CF_DNS_API_TOKEN}
    restart: unless-stopped
```

---

## 3. Environment Variables (`.env`)

Configure your DNS API variables locally in a `.env` file. These values will be automatically injected into your compose configuration when executing:

```ini
# DNS-01 solver configuration
DNS_PROVIDER=cloudflare
CF_DNS_API_TOKEN=your_private_cloudflare_dns_api_token
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
