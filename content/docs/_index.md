---
title: "Welcome to Certer"
description: "What is Certer?"
icon: "menu_book"
weight: 10
---

**Certer** is a lightweight, containerized certificate manager designed to automate the lifecycle of SSL/TLS certificates. It integrates directly with ACME providers like Let's Encrypt and ZeroSSL using the Lego library, solving DNS-01 and HTTP-01 challenges seamlessly.

## Core Features

- **Automated Renewals**: Schedules expiration checks and renews certificates automatically in the background.
- **REST API**: Simple JSON endpoints to fetch issued certificate chains and private keys.
- **Argon2id Authentication**: Secure, constant-time token verification for client and admin endpoints.
- **Multi-Tenant Scoping**: Fine-grained team-based controls for API keys and certificates.
