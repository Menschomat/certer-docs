---
title: "Docker Run Quickstart"
description: "Start the Certer daemon on a single host using Docker CLI."
icon: "terminal"
weight: 210
---

Running Certer via the Docker CLI is the fastest way to get a daemon up and running.

## Prerequisites

Before starting, ensure you have:
*   [Docker](https://docs.docker.com/get-docker/) installed.
*   An active domain configuration (e.g. on Cloudflare) with DNS zone control.

---

## Step 1: Generate API Token Hash

Certer secures config updates and certificate fetching using Argon2id token hashes. Instead of writing plaintext tokens to the configuration file, we store their cryptographic hash.

Run the built-in `keygen` utility inside the Certer container to generate a secure hash:

```bash
docker run --rm m0space/certer:latest keygen -token "my_super_secret_token"
```

**Output Example:**
```text
Token Hash (Argon2id): $argon2id$v=19$m=65536,t=3,p=2$UnVxYWp4djNhSDBhN1E5Nw$8d1S5G34vH23c...
```
> [!IMPORTANT]
> Save the returned hash. You will need to paste it into your `config.json` configuration file. Keep your cleartext token `"my_super_secret_token"` safe as you will pass it in client API headers.

---

## Step 2: Create a Minimal Configuration (`config.json`)

Create a `config.json` file in your current working directory to define your first Team and register your generated API token:

```json
{
  "teams": [
    {
      "id": "019eebb8-74a1-70da-96fb-1d2d28db29b9",
      "name": "Production Operations",
      "description": "Production services certificate owners"
    }
  ],
  "api_keys": [
    {
      "id": "019eebb8-74a1-70da-96fb-1d2d28db29c0",
      "description": "admin-token",
      "token_hash": "$argon2id$v=19$m=65536,t=3,p=2$UnVxYWp4djNhSDBhN1E5Nw$8d1S5G34vH23c...",
      "allowed_teams": [],
      "allowed_certificates": [],
      "admin": true
    }
  ],
  "certificates": []
}
```

---

## Step 3: Run the Container

Start the Certer daemon by binding standard ports and mounting both the config file and a directory to store issued certificates:

```bash
docker run -d \
  --name certer \
  -p 8080:8080 \
  -v $(pwd)/certs:/app/certs \
  -v $(pwd)/config.json:/app/config.json \
  -e CLOUDFLARE_EMAIL="admin@yourdomain.com" \
  -e CLOUDFLARE_API_KEY="your_cloudflare_api_token" \
  -e LOG_LEVEL="info" \
  -e LOG_STYLE="json" \
  m0space/certer:latest
```

### Verified Run Parameters:
*   `-p 8080:8080`: Exposes the REST API on port `8080`.
*   `-v ...:/app/certs`: Mounts a folder where Let's Encrypt certificates and private key files will be written.
*   `-v ...:/app/config.json`: Mounts the configuration schema.
*   `-e CLOUDFLARE_...`: Configures the DNS-01 solver validation variables.

---

## Step 4: Verify Logging

Verify the container logs to ensure it initialized the background scheduler loop:

```bash
docker logs -f certer
```

If successful, you will see a JSON-style log entry indicating the scheduler has started checking configured certificate lifecycles.
