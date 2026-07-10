---
title: "Audit Reports"
description: "Generate static audit reports from the Certer API."
icon: "monitoring"
weight: 440
---

The `audit` CLI queries Certer with an admin API key and renders an inventory of teams, certificate configurations, API keys, and certificate health.

For a static dashboard, generate an HTML report:

```bash
AUDIT_TOKEN=your-admin-token /audit -url http://localhost:8080 -format html -output certer-report.html
```

The HTML report is a standalone file with embedded CSS. It uses `GET /api/v1/certificates/status`, so it only transfers certificate health metadata and does not download certificate PEM bodies or private keys.

Supported formats:

| Format | Description |
|---|---|
| `text` | Markdown report printed to stdout. |
| `json` | Raw audit data as formatted JSON. |
| `html` | Static dashboard for operators. |

The CLI also supports environment variables:

| Variable | Description |
|---|---|
| `AUDIT_TOKEN` | Admin API token used for authenticated API requests. |
| `AUDIT_URL` | Target Certer API URL. |
| `AUDIT_FORMAT` | Output format: `text`, `json`, or `html`. |
| `AUDIT_OUTPUT` | Optional file path for writing the report. |
