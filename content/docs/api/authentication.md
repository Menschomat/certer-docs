---
title: "Authentication"
description: "How to authenticate requests using scoped API tokens."
icon: "vpn_key"
weight: 410
---

All protected Certer REST API requests must include a valid Authorization token header. The `/health` and `/api/v1/hello` endpoints are public.

## Authorization Header

Pass the cleartext API key inside the HTTP standard header:

```http
Authorization: Bearer <your_cleartext_api_token>
```

> [!CAUTION]
> Plaintext tokens are only returned once by the server on key creation. If you lose the token, you must delete the API key configuration and generate a new one.

---

## Token Verification Details

When a request arrives, the server processes token verification as follows:

1.  **Extract**: The Bearer token is parsed from the HTTP header.
2.  **Argon2id KDF**: The server checks the key configurations and hashes the request token using the Argon2id key derivation function.
3.  **Constant-Time Comparison**: The hash of the request token is verified against the stored hash in `config.json` using `crypto/subtle.ConstantTimeCompare` to avoid token-dependent comparison timing.

### Argon2id Parameters

Certer uses the following hashing configuration to ensure cryptographic strength:
*   **Memory**: `65536 KB`
*   **Iterations (Time)**: `3`
*   **Parallelism (Threads)**: `2`
*   **Salt**: Random 16-byte cryptographically secure (`crypto/rand`).

---

## Error Status Codes

If validation fails, the API returns standard HTTP error codes:

| Status Code | Description |
|---|---|
| `401 Unauthorized` | The `Authorization` header is missing, malformed, or the token is invalid. |
| `403 Forbidden` | The token is valid, but is not scoped to access the requested Team, API Key, or Certificate. |
