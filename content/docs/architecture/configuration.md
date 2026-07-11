---
title: "Configuration Reference"
description: "Reference guide for Certer environment variables and configuration files."
icon: "settings_suggest"
weight: 320
---

Certer supports configuration using both **Environment Variables** (for runtime secrets, database locations, logging) and a **`config.json`** file (defining teams, keys, and certificates).

---

## 1. Configuration Options

| JSON field | Type | Default | Environment variable | Description |
|---|---|---|---|---|
| `port` | string | `"8080"` | `PORT` | Port for the HTTP API server. |
| `https_port` | string | `"8443"` | `HTTPS_PORT` | Port for the HTTPS API server. |
| `ssl_cert_id` | string | *(none)* | *(none)* | Optional certificate configuration ID used by the HTTPS listener. |
| `env` | string | `"development"` | `ENV` | Service environment. Use `development` or `production`. |
| `acme_provider` | string | `"letsencrypt"` | `ACME_PROVIDER` | ACME provider. Supported values are `letsencrypt` and `zerossl`. |
| `acme_directory_url` | string | *(dynamic)* | `ACME_DIRECTORY_URL` | ACME directory URL. If omitted, Certer selects the provider default for the configured environment. |
| `acme_email` | string | *(none)* | `ACME_EMAIL` | Email address registered with the ACME provider. Required for the renewal scheduler. |
| `eab_kid` | string | *(none)* | `EAB_KID` | External Account Binding key ID, used for manual ZeroSSL EAB configuration. |
| `eab_hmac` | string | *(none)* | `EAB_HMAC` | External Account Binding HMAC, used for manual ZeroSSL EAB configuration. |
| `cert_storage_dir` | string | `"./certs"` | `CERT_STORAGE_DIR` | Directory where certificates and private keys are persisted. |
| `challenge_port` | string | `"5002"` | `CHALLENGE_PORT` | HTTP port for the HTTP-01 challenge solver. |
| `dns_provider` | string | *(none)* | `DNS_PROVIDER` | DNS provider name, such as `cloudflare`, `hetzner`, `route53`, or any Lego-supported provider. Blank falls back to HTTP-01. |
| `dns_resolvers` | list | *(none)* | `DNS_RESOLVERS` | DNS resolvers used to verify DNS-01 propagation. The environment variable is comma-separated. |
| `renew_threshold_days` | int | `30` | `RENEW_THRESHOLD_DAYS` | Days before expiry when Certer should renew automatically. |
| `check_interval_hours` | int | `24` | `CHECK_INTERVAL_HOURS` | Hours between local certificate status checks. |
| `teams` | list | *(none)* | *(none)* | Static team metadata. |
| `certificates` | list | *(none)* | *(none)* | Static certificate definitions. |
| `api_keys` | list | *(none)* | *(none)* | Static API key definitions. |

Additional loader paths are configured only by environment variable:

| Variable | Default | Description |
|---|---|---|
| `CONFIG_PATH` | `./config.json` | Path to the static JSON configuration file. |
| `STATE_PATH` | `state.json` next to `CONFIG_PATH` | Path to the runtime mutable state file used for dynamically managed teams, certificates, and API keys. |

---

## 2. Configuration Schema (`config.json`)

The config file defines the static system state. Static teams, API keys, and certificates must include explicit `id` values. Runtime-created items are stored separately in `state.json`, where Certer can assign missing IDs and merge them with the static configuration at runtime.

### JSON Field Reference

#### Root Fields
*   `teams` (array of teams): Registered administrative organizations.
*   `api_keys` (array of API keys): Active API tokens scoped to operations.
*   `certificates` (array of certificates): Domains managed by the background solver.
*   Runtime options from the table above, such as `port`, `https_port`, `acme_email`, `dns_provider`, and `renew_threshold_days`.

#### Team Object
*   `id` (string): Unique identifier.
*   `name` (string): Name of the team.
*   `description` (string): Description of ownership.

#### API Key Object
*   `id` (string): Unique identifier.
*   `description` (string): User-friendly description of token scope.
*   `token` (string): **Argon2id** hash of the token. Plaintext tokens are never stored.
*   `allowed_teams` (array of strings): Team IDs this key is scoped to.
*   `allowed_certificates` (array of strings): Certificate IDs a standard token is authorized to read.
*   `admin` (boolean): If `true`, this is a configuration/control-plane key. If `false`, it is a fetch key limited to retrieving raw certificates.

#### Certificate Object
*   `id` (string): Unique identifier.
*   `team_id` (string): Owning team ID. If omitted, Certer coerces the certificate to the built-in `system` team.
*   `primary` (string): The primary domain name, for example `example.com`.
*   `sans` (array of strings): Subject Alternative Names, for example `*.example.com` or `www.example.com`.
*   `description` (string): Human-readable certificate purpose.
*   `dns_provider` (string, optional): Per-certificate DNS provider override. If omitted, Certer uses the root `dns_provider`.

---

## 3. Listener and ACME Behavior

Certer starts both an HTTP listener and an HTTPS listener by default:

*   HTTP listens on `port`, defaulting to `8080`.
*   HTTPS listens on `https_port`, defaulting to `8443`.
*   If `ssl_cert_id` is configured and matching certificate files exist in `cert_storage_dir`, Certer uses that managed certificate for HTTPS.
*   If no matching HTTPS certificate exists, Certer generates an in-memory self-signed P-256 ECDSA certificate so the HTTPS listener can start immediately.

For Let's Encrypt, `acme_directory_url` is selected automatically when omitted:

*   `ENV=development` uses Let's Encrypt staging.
*   `ENV=production` uses Let's Encrypt production.

For ZeroSSL, set `acme_provider` to `zerossl`. Certer can either register with ZeroSSL using `acme_email`, or use manual EAB credentials through `eab_kid` / `eab_hmac`.

DNS-01 uses the Lego DNS provider named by `dns_provider`. Provider-specific credentials are read using Lego's environment variables. For example:

```sh
export DNS_PROVIDER="cloudflare"
export DNS_RESOLVERS="1.1.1.1:53,8.8.8.8:53"
export CF_DNS_API_TOKEN="your_cloudflare_token"

export DNS_PROVIDER="hetzner"
export HETZNER_API_TOKEN="your_hetzner_api_token"
```

---

### Example Configuration

```json
{
  "acme_email": "admin@example.com",
  "dns_provider": "cloudflare",
  "dns_resolvers": [
    "1.1.1.1:53",
    "8.8.8.8:53"
  ],
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
      "token": "$argon2id$v=19$m=65536,t=3,p=2$UnVxYWp4djNhSDBhN1E5Nw$8d1S5G34vH23c...",
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
      "description": "Production wildcard certificate"
    }
  ]
}
```
