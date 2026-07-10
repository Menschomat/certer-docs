---
title: "Fetch Certificates"
description: "REST endpoints for fetching raw public certificate chains and private keys."
icon: "download"
weight: 430
---

These endpoints are used by standard clients (proxies, load balancers, cron tasks) to retrieve the raw PEM-encoded certificate assets.

They require a Bearer token mapped to a standard fetch API key (`admin = false`) authorized to access the targeted team or certificate.

---

## Get Active Certificates

Retrieve the full list of certificates the authenticated token is authorized to fetch.

*   **Path**: `GET /api/v1/certificates`
*   **Headers**: `Authorization: Bearer <fetch_token>`
*   **Response (`200 OK`)**:
    ```json
    [
      {
        "certificate_id": "019eebb8-74a1-70da-96fb-1d2d28db29d0",
        "domain": "example.com",
        "sans": [
          "*.example.com"
        ],
        "issued": true,
        "certificate": "-----BEGIN CERTIFICATE-----\nMIIFdTCCBFWgAwIBAgISA2J...\n-----END CERTIFICATE-----",
        "private_key": "-----BEGIN EC PRIVATE KEY-----\nMHQCAQEEIB4...\n-----END EC PRIVATE KEY-----",
        "cert_filename": "019eebb8-74a1-70da-96fb-1d2d28db29d0.crt",
        "key_filename": "019eebb8-74a1-70da-96fb-1d2d28db29d0.key"
      }
    ]
    ```

---

## Get Certificate Status Without PEM Material

Use these endpoints for monitoring, audit reports, dashboards, and health checks that need certificate status without transferring certificate bodies or private keys.

*   **Paths**:
    *   `GET /api/v1/certificates/status`
    *   `GET /api/v1/certificates/{identifier}/status`
*   **Headers**: `Authorization: Bearer <fetch_token_or_admin_token>`
*   **Response (`200 OK`)**:
    ```json
    {
      "id": "019eebb8-74a1-70da-96fb-1d2d28db29d0",
      "domain": "example.com",
      "sans": [
        "*.example.com"
      ],
      "issued": true,
      "status": "ok",
      "issued_at": "2026-01-01T00:00:00Z",
      "expires_at": "2026-04-01T00:00:00Z",
      "days_remaining": 72,
      "is_valid": true,
      "issuer_common_name": "R3",
      "serial_number": "5f2b8c..."
    }
    ```

The `status` value is `ok`, `warning`, or `critical`. A certificate is marked `warning` when it has 30 days or fewer remaining, and `critical` when it is expired, not currently valid, missing certificate/key files, or cannot be parsed.

---

## Get Raw PEM Files (No JSON)

Directly download raw PEM blocks for a public certificate or private key. Handy for simple scripts (`curl`) that do not have `jq` or Python parsing capabilities.

### Get Raw Certificate
*   **Path**: `GET /api/v1/certificates/{identifier}/certificate`
*   **Headers**: `Authorization: Bearer <fetch_token>`
*   **Path Parameter**: `{identifier}` can be the Certificate UUID or a domain name (matches primary or SAN wildcard).
*   **Response (`200 OK`)**: `text/plain` payload
    ```text
    -----BEGIN CERTIFICATE-----
    MIIFdTCCBFWgAwIBAgISA2J...
    -----END CERTIFICATE-----
    ```

### Get Raw Private Key
*   **Path**: `GET /api/v1/certificates/{identifier}/private-key`
*   **Headers**: `Authorization: Bearer <fetch_token>`
*   **Path Parameter**: `{identifier}` can be the Certificate UUID or a domain name (matches primary or SAN wildcard).
*   **Response (`200 OK`)**: `text/plain` payload
    ```text
    -----BEGIN EC PRIVATE KEY-----
    MHQCAQEEIB4...
    -----END EC PRIVATE KEY-----
    ```

### Handling Duplicate/Ambiguous Domains
If `{identifier}` is a domain name that matches multiple certificate configurations:
1. **Primary Domain Wins**: The endpoint prioritizes matching the primary domain over SANs.
2. **Chronological Tie-Breaker (UUIDv7)**: If multiple configurations still match, the newest configuration (highest UUIDv7 value) is returned.
3. **Explicit ID**: You can bypass ambiguity by passing the explicit Certificate UUID instead of the domain.

---

## Scoping & Access Control

When executing `GET /api/v1/certificates`, Certer inspects the request token claims and filters the return payload:

1.  **Allowed Teams check**: If the API key has `allowed_teams` specified (e.g. `["team-A-uuid"]`), the request will *only* return certificates where `team_id = "team-A-uuid"`.
2.  **Allowed Certificates check**: If the API key has `allowed_certificates` specified (e.g. `["cert-1-uuid"]`), the request will *only* return the matching certificate configuration.
3.  **Issuance status check**: Only configurations where `"issued": true` and the corresponding PEM files exist in storage are returned with raw PEM bodies. If a certificate configuration exists but has not yet completed ACME negotiation, it is skipped or returned with empty values.
