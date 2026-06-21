---
title: "Configuration Reference"
description: "Reference guide for Certer environment variables and configuration files."
icon: "settings_suggest"
weight: 320
---

Certer supports configuration using both **Environment Variables** (for runtime secrets, database locations, logging) and a **`config.json`** file (defining teams, keys, and certificates).

---

## 1. Environment Variables

| Variable | Default | Description |
|---|---|---|
| `PORT` | `8080` | Port the API server listens on. |
| `CONFIG_PATH` | `/app/config.json` | Path to the JSON configuration file. |
| `CERTS_DIR` | `/app/certs` | Directory where raw certificates and keys are saved. |
| `LOG_LEVEL` | `info` | Structured logging verbosity (`debug`, `info`, `warn`, `error`). |
| `LOG_STYLE` | `json` | Logging format style. Set to `json` for machine-friendly structured logging, or `text` for readable development output. |
| `CLOUDFLARE_EMAIL` | *(Optional)* | Email address associated with your Cloudflare account (for DNS-01 validation). |
| `CLOUDFLARE_API_KEY` | *(Optional)* | Cloudflare API Token/Key for managing DNS records. |

---

## 2. Configuration Schema (`config.json`)

The config file represents the system state. On startup, if any configuration items (Teams, Keys, Certificates) lack a `id` property, Certer automatically generates and inserts a **UUIDv7** string.

### JSON Field Reference

#### Root Fields
*   `teams` (Array of Teams): Registered administrative organizations.
*   `api_keys` (Array of API Keys): Active API tokens scoped to operations.
*   `certificates` (Array of Certificates): Domains managed by the background solver.

#### Team Object
*   `id` (String, UUIDv7): Unique identifier.
*   `name` (String): Name of the team.
*   `description` (String): Description of ownership.

#### API Key Object
*   `id` (String, UUIDv7): Unique identifier.
*   `description` (String): User-friendly description of token scope.
*   `token_hash` (String): **Argon2id** hash of the token. Plaintext tokens are never stored.
*   `allowed_teams` (Array of Strings): UUIDs of teams this key is scoped to. (An empty array permits all teams for admin keys).
*   `allowed_certificates` (Array of Strings): Certificate UUIDs a standard token is authorized to read.
*   `admin` (Boolean): If `true`, this is a configuration/control-plane key. If `false`, it is a fetch key limited to retrieving raw certificates.

#### Certificate Object
*   `id` (String, UUIDv7): Unique identifier.
*   `team_id` (String, UUIDv7): Owning team ID.
*   `primary` (String): The primary domain name (e.g. `example.com`).
*   `sans` (Array of Strings): Subject Alternative Names (e.g. `*.example.com`, `www.example.com`).
*   `issued` (Boolean): Managed by the scheduler, indicates if Let's Encrypt has generated active certificate files.

---

### Example Configuration

```json
{
  "teams": [
    {
      "id": "019eebb8-74a1-70da-96fb-1d2d28db29b9",
      "name": "Platform Operations",
      "description": "Manages public endpoints"
    }
  ],
  "api_keys": [
    {
      "id": "019eebb8-74a1-70da-96fb-1d2d28db29c0",
      "description": "cluster-admin-token",
      "token_hash": "$argon2id$v=19$m=65536,t=3,p=2$UnVxYWp4djNhSDBhN1E5Nw$8d1S5G34vH23c...",
      "allowed_teams": [],
      "allowed_certificates": [],
      "admin": true
    }
  ],
  "certificates": [
    {
      "id": "019eebb8-74a1-70da-96fb-1d2d28db29d0",
      "team_id": "019eebb8-74a1-70da-96fb-1d2d28db29b9",
      "primary": "example.com",
      "sans": [
        "*.example.com"
      ],
      "issued": true
    }
  ]
}
```
