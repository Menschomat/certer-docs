---
title: "Provider Setup"
description: "How to register and configure the Certer provider in your Terraform files."
icon: "construction"
weight: 510
---

The Certer Terraform provider is published on the official registry.

## 1. Registry Requirements

Add the provider definition to your root module's `required_providers` block:

```hcl
terraform {
  required_providers {
    certer = {
      source  = "Menschomat/certer"
      version = "~> 1.0"
    }
  }
}
```

---

## 2. Provider Block Configuration

Configure the provider with the endpoint address and an administrative API key (`admin = true`):

```hcl
provider "certer" {
  address = "https://certer.example.com:8080"
  token   = "your_admin_api_token"
}
```

### Argument Reference
*   `address` (String, Required): The HTTP/HTTPS endpoint of your Certer server deployment. Can also be set via the `CERTER_ADDRESS` environment variable.
*   `token` (String, Required): An administrative Bearer token. Can also be set via the `CERTER_TOKEN` environment variable.

---

## 3. Local Installation (For Development / Offline)

For local testing or private air-gapped systems, you can compile and register the provider locally.

1.  Clone and compile the binary:
    ```bash
    git clone https://github.com/Menschomat/terraform-provider-certer.git
    cd terraform-provider-certer
    go build -o terraform-provider-certer
    ```
2.  Move it to your local plugin directory matching your architecture (e.g. macOS Apple Silicon):
    ```bash
    mkdir -p ~/.terraform.d/plugins/registry.terraform.io/Menschomat/certer/1.0.0/darwin_arm64/
    cp terraform-provider-certer ~/.terraform.d/plugins/registry.terraform.io/Menschomat/certer/1.0.0/darwin_arm64/
    ```
3.  Execute `terraform init` to resolve the local provider copy.
