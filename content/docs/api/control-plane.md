---
title: "Control Plane APIs"
description: "REST endpoints for managing configurations, teams, and API keys."
icon: "admin_panel_settings"
weight: 420
---

These endpoints are used to manage configurations. They require a Bearer token mapped to an API key where `admin = true`.

Static resources loaded from `config.json` can be read through these endpoints, but only runtime resources stored in `STATE_PATH` can be updated or deleted. Scoped admin keys can manage certificates and API keys only inside their allowed teams. Team creation, update, and deletion require an unscoped root admin key.

---

## 1. Teams Endpoints

Manage team organizations.

### Get All Teams
*   **Path**: `GET /api/v1/config/teams`
*   **Headers**: `Authorization: Bearer <admin_token>`
*   **Response (`200 OK`)**:
    ```json
    [
      {
        "id": "019eebb8-74a1-70da-96fb-1d2d28db29b9",
        "name": "Production Operations",
        "description": "Production services"
      }
    ]
    ```

### Create a Team
*   **Path**: `POST /api/v1/config/teams`
*   **Request Body**:
    ```json
    {
      "name": "Beta Team",
      "description": "Staging environments"
    }
    ```
*   **Response (`201 Created`)**:
    Returns the created team including the automatically generated UUIDv7.

### Update a Team
*   **Path**: `PUT /api/v1/config/teams/{id}`
*   **Request Body**:
    ```json
    {
      "name": "Beta Team",
      "description": "Edge and staging environments"
    }
    ```
*   **Response (`200 OK`)**:
    Returns the updated team.

### Delete a Team
*   **Path**: `DELETE /api/v1/config/teams/{id}`
*   **Response (`204 No Content`)**: Success.
*   **Response (`400 Bad Request`)**: Returns `400` if the team is currently associated with active certificates or scoped API keys (referential integrity check).

---

## 2. API Keys Endpoints

Manage administrative and client credentials.

### Get API Keys
*   **Path**: `GET /api/v1/config/api_keys`
*   **Response (`200 OK`)**:
    ```json
    [
      {
        "id": "019eebb8-74a1-70da-96fb-1d2d28db29c0",
        "token": "$argon2id$v=19$m=65536,t=3,p=2$...",
        "description": "production-fetcher-key",
        "allowed_teams": ["019eebb8-74a1-70da-96fb-1d2d28db29b9"],
        "allowed_certificates": [],
        "admin": false
      }
    ]
    ```

### Create an API Key
*   **Path**: `POST /api/v1/config/api_keys`
*   **Request Body**:
    ```json
    {
      "description": "production-fetcher-key",
      "allowed_teams": ["019eebb8-74a1-70da-96fb-1d2d28db29b9"],
      "allowed_certificates": [],
      "admin": false
    }
    ```
*   **Response (`201 Created`)**:
    ```json
    {
      "id": "019eebb8-74a1-70da-96fb-1d2d28db29c0",
      "description": "production-fetcher-key",
      "allowed_teams": ["019eebb8-74a1-70da-96fb-1d2d28db29b9"],
      "allowed_certificates": [],
      "admin": false,
      "token": "$argon2id$v=19$m=65536,t=3,p=2$...",
      "cleartext_token": "cert_token_46ab34fd9128de..."
    }
    ```
    > [!IMPORTANT]
    > The `cleartext_token` value is only returned once during creation. The `token` field is the stored Argon2id hash.

### Update an API Key
*   **Path**: `PUT /api/v1/config/api_keys/{id}`
*   **Request Body**:
    ```json
    {
      "description": "production-fetcher-key",
      "allowed_teams": ["019eebb8-74a1-70da-96fb-1d2d28db29b9"],
      "allowed_certificates": [],
      "admin": false
    }
    ```
*   **Response (`200 OK`)**:
    Returns the updated API key configuration. Updating an API key does not rotate its token.

### Delete an API Key
*   **Path**: `DELETE /api/v1/config/api_keys/{id}`
*   **Response (`204 No Content`)**

---

## 3. Certificate Configurations

Manage domains that Certer should monitor and renew.

### Create or Update a Certificate Configuration
*   **Paths**:
    *   `POST /api/v1/config/certificates`
    *   `PUT /api/v1/config/certificates/{id}`
*   **Request Body**:
    ```json
    {
      "team_id": "019eebb8-74a1-70da-96fb-1d2d28db29b9",
      "primary": "example.com",
      "sans": [
        "*.example.com",
        "www.example.com"
      ],
      "description": "Production wildcard certificate",
      "dns_provider": "cloudflare"
    }
    ```
*   **Response (`200 OK` or `201 Created`)**

### Delete a Certificate Configuration
*   **Path**: `DELETE /api/v1/config/certificates/{id}`
*   **Response (`204 No Content`)**
    *   *Note: Deleting a configuration removes it from the renewal scheduler. The generated PEM certificate files in the cert storage are kept intact.*
