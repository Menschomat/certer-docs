---
title: "Data Sources Reference"
description: "Retrieve issued certificates, keys, and parsed x509 validity metrics."
icon: "find_in_page"
weight: 530
---

The `certer_certificate_data` data source is used to fetch raw certificate PEM chains and private keys, as well as parsed cryptographic metrics.

> [!NOTE]
> Fetching certificate assets inside a Data Source instead of using resource attributes prevents **inconsistent plan errors**. Because Certer renews certificates in the background, updating values within resource attributes directly would cause state mismatches during Terraform runs. Using a Data Source handles background renewals cleanly.

---

## Example Usage

Retrieve an issued certificate and pass its assets to a cloud provider (e.g. AWS ACM):

```hcl
data "certer_certificate_data" "wildcard" {
  certificate_id = "019eebb8-74a1-70da-96fb-1d2d28db29d0"
}

# Import to AWS ACM
resource "aws_acm_certificate" "imported" {
  private_key       = data.certer_certificate_data.wildcard.private_key
  certificate_body  = data.certer_certificate_data.wildcard.certificate
}
```

---

## Attribute Reference

All attributes below (except `certificate_id`) are read-only (`Computed`). If Certer has not yet issued the certificate, computed fields return `null`.

| Attribute | Type | Description |
|---|---|---|
| `certificate_id` | String | **(Required)** The configuration UUID of the certificate. |
| `domain` | String | The primary domain name. |
| `sans` | List of Strings | Subject Alternative Names (SANs). |
| `issued` | Boolean | `true` if the certificate is actively generated on the server. |
| `certificate` | String | The PEM-encoded certificate chain (Sensitive). |
| `private_key` | String | The PEM-encoded private key (Sensitive). |
| `cert_filename` | String | Suggested certificate download filename returned by Certer. |
| `key_filename` | String | Suggested key download filename returned by Certer. |
| `issued_at` | String | The start of validity (`NotBefore`) in RFC3339 format. |
| `expires_at` | String | The end of validity (`NotAfter`) in RFC3339 format. |
| `days_remaining` | Integer | Number of days remaining before certificate expiration. |
| `is_valid` | Boolean | `true` if the current time is within validity bounds. |
| `serial_number` | String | Hexadecimal representation of the certificate serial number. |
| `issuer_common_name`| String | The Common Name (CN) of the issuing authority. |
| `signature_algorithm`| String | Signature algorithm name (e.g. `SHA256-RSA`, `ECDSA-SHA256`). |
| `key_algorithm` | String | Public key algorithm type (e.g. `RSA`, `ECDSA`). |
| `key_size` | Integer | Key size in bits (e.g., `2048`, `256`). |
