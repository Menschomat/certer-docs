---
title: "Keygen"
description: "Generate Argon2id token hashes for Certer API keys."
icon: "key"
weight: 10
---

Certer stores API tokens as Argon2id hashes. Use the `keygen` utility to generate a hash for a cleartext token before adding it to `config.json`.

## Docker Image

Run `keygen` from the published container image:

```bash
docker run --rm --entrypoint /keygen ghcr.io/menschomat/certer:latest -token "my_super_secret_token"
```

Example output:

```text
Token Hash (Argon2id): $argon2id$v=19$m=65536,t=3,p=2$UnVxYWp4djNhSDBhN1E5Nw$8d1S5G34vH23c...
```

Keep the cleartext token somewhere safe. Certer needs the hash in configuration, while clients send the cleartext token in the `Authorization: Bearer ...` header.

## Local Checkout

From the Certer repository, you can also run the helper script:

```bash
./hash.sh -token mysecret
```

Without `-token`, the helper generates a random token and its hash.
