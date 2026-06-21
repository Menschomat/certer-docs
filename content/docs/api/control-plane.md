---
title: "Control Plane APIs"
description: "REST endpoints for managing configurations, teams, and API keys."
icon: "admin_panel_settings"
weight: 420
---

These endpoints are used to manage configurations. They require a Bearer token mapped to an API key where `admin = true`.

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

### Delete a Team
*   **Path**: `DELETE /api/v1/config/teams/{id}`
*   **Response (`204 No Content`)**: Success.
*   **Response (`400 Bad Request`)**: Returns `400` if the team is currently associated with active certificates or scoped API keys (referential integrity check).

---

## 2. API Keys Endpoints

Manage administrative and client credentials.

### Create an API Key
*   **Path**: `POST /api/v1/config/api-keys`
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
      "cleartext_token": "cert_token_46ab34fd9128de..."
    }
    ```
    > [!IMPORTANT]
    > The `cleartext_token` parameter is only returned once during creation.

### Delete an API Key
*   **Path**: `DELETE /api/v1/config/api-keys/{id}`
*   **Response (`204 No Content`)**

---

## 3. Certificate Configurations

Manage domains that Certer should monitor and renew.

### Create or Update a Certificate Configuration
*   **Path**: `PUT /api/v1/config/certificates/{id}` or `POST /api/v1/config/certificates`
*   **Request Body**:
    ```json
    {
      "team_id": "019eebb8-74a1-70da-96fb-1d2d28db29b9",
      "primary": "example.com",
      "sans": [
        "*.example.com",
        "www.example.com"
      ]
    }
    ```
*   **Response (`200 OK` or `201 Created`)**

### Delete a Certificate Configuration
*   **Path**: `DELETE /api/v1/config/certificates/{id}`
*   **Response (`204 No Content`)**
    *   *Note: Deleting a configuration removes it from the renewal scheduler. The generated PEM certificate files in the cert storage are kept intact.*
