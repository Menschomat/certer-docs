---
title: "Kubernetes Deployment"
description: "Deploy Certer to a Kubernetes cluster using standard manifests."
icon: "cloud"
weight: 230
---

In a Kubernetes environment, Certer is run as a single-replica `Deployment` or `StatefulSet` with a persistent volume to store issued SSL/TLS certificates.

---

## 1. Secrets Configuration

Store your Cloudflare DNS-01 API token securely inside a Kubernetes `Secret`:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: certer-credentials
  namespace: certer
type: Opaque
stringData:
  cloudflare-email: "admin@yourdomain.com"
  cloudflare-api-key: "your_cloudflare_api_token"
```

---

## 2. Configuration Map (`config.json`)

Deploy a `ConfigMap` containing your initial Teams and API keys:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: certer-config
  namespace: certer
data:
  config.json: |
    {
      "teams": [
        {
          "id": "019eebb8-74a1-70da-96fb-1d2d28db29b9",
          "name": "Production Operations",
          "description": "Production services"
        }
      ],
      "api_keys": [
        {
          "id": "019eebb8-74a1-70da-96fb-1d2d28db29c0",
          "description": "admin-token",
          "token_hash": "$argon2id$v=19$m=65536,t=3,p=2$UnVxYWp4djNhSDBhN1E5Nw$8d1S5G34vH23c...",
          "allowed_teams": [],
          "allowed_certificates": [],
          "admin": true
        }
      ],
      "certificates": []
    }
```

---

## 3. Persistent Volume Claim (PVC)

Because Let's Encrypt enforces rate limits on key generation and signature requests, it is critical to persist the private keys and issued certificates across pod restarts:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: certer-certs-pvc
  namespace: certer
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

---

## 4. Deployment Manifest

Deploy Certer mounting both the configuration and certificate files. Since ACME verification is run on a single background scheduler loop, we limit deployment replicas to `1` to avoid concurrency collisions:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: certer
  namespace: certer
  labels:
    app: certer
spec:
  replicas: 1 # Strictly 1 to avoid ACME double-signing locks
  selector:
    matchLabels:
      app: certer
  template:
    metadata:
      labels:
        app: certer
    spec:
      containers:
        - name: certer
          image: m0space/certer:latest
          ports:
            - containerPort: 8080
              name: http-api
          env:
            - name: CLOUDFLARE_EMAIL
              valueFrom:
                secretKeyRef:
                  name: certer-credentials
                  key: cloudflare-email
            - name: CLOUDFLARE_API_KEY
              valueFrom:
                secretKeyRef:
                  name: certer-credentials
                  key: cloudflare-api-key
            - name: LOG_LEVEL
              value: "info"
            - name: LOG_STYLE
              value: "json"
          volumeMounts:
            - name: certs-vol
              mountPath: /app/certs
            - name: config-vol
              mountPath: /app/config.json
              subPath: config.json
      volumes:
        - name: certs-vol
          persistentVolumeClaim:
            claimName: certer-certs-pvc
        - name: config-vol
          configMap:
            name: certer-config
---
apiVersion: v1
kind: Service
metadata:
  name: certer
  namespace: certer
spec:
  ports:
    - port: 8080
      targetPort: 8080
      name: http
  selector:
    app: certer
```
