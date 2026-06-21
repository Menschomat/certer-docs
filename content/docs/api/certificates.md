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

## Scoping & Access Control

When executing `GET /api/v1/certificates`, Certer inspects the request token claims and filters the return payload:

1.  **Allowed Teams check**: If the API key has `allowed_teams` specified (e.g. `["team-A-uuid"]`), the request will *only* return certificates where `team_id = "team-A-uuid"`.
2.  **Allowed Certificates check**: If the API key has `allowed_certificates` specified (e.g. `["cert-1-uuid"]`), the request will *only* return the matching certificate configuration.
3.  **Issuance status check**: Only configurations where `"issued": true` and the corresponding PEM files exist in storage are returned with raw PEM bodies. If a certificate configuration exists but has not yet completed ACME negotiation, it is skipped or returned with empty values.
