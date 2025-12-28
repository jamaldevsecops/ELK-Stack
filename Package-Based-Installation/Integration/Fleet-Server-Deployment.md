# 🚀 Fleet Server Deployment Guide (Self‑Managed Elastic Stack)

This guide provides **step‑by‑step instructions** to deploy **Fleet Server** in a **self‑managed Elasticsearch + Kibana environment** with **TLS enabled**, based on real production troubleshooting and best practices.

---

## 🧭 Architecture Overview

Elastic Agents → Fleet Server (Elastic Agent) → Elasticsearch Cluster  
                                     ↘ Kibana (Fleet UI)

Fleet Server acts as the **control plane** for Elastic Agents.

---

## ✅ Prerequisites

### Infrastructure
- Elasticsearch cluster (≥ 1 healthy node)
- Kibana (Fleet enabled)
- Linux host for Fleet Server
- Static IP / DNS for Fleet Server

### Software Versions
- Elasticsearch ≥ 8.x
- Kibana ≥ 8.x
- Elastic Agent ≥ same version

### Network
- Fleet Server port: **8220**
- Elasticsearch port: **9200**
- Kibana port: **5601**

---

## 🔐 Certificate Strategy (Recommended)

Use a **single internal CA** for:
- Elasticsearch HTTP
- Fleet Server HTTPS

You will need:
- `ca.crt`
- `fleet-server.crt`
- `fleet-server.key`

> ❗ Agents **must trust the CA** during enrollment.

---

## 🧩 Step 1: Configure Fleet in Kibana

### Enable Fleet (if not already)
```yaml
xpack.fleet.enabled: true
```

Restart Kibana.

---

## 🧩 Step 2: Configure Fleet Settings (UI)

Navigate to:
**Kibana → Management → Fleet → Settings**

### Fleet Server Hosts
```
https://192.168.20.128:8220
```

### Outputs (Elasticsearch)
```
https://es1:9200
https://es2:9200
https://es3:9200
```

### TLS (Optional but recommended)
Paste:
- Elasticsearch CA
- Fleet Server cert & key (for reference)

> ℹ️ These are **metadata**, not file distribution.

Save settings.

---

## 🧩 Step 3: Create Fleet Server Policy

Fleet → Agent Policies → Create policy  
Name: `fleet-server-policy`  
Enable **Fleet Server integration**.

---

## 🧩 Step 4: Generate Service Token

On any Elasticsearch node:
```bash
/usr/share/elasticsearch/bin/elasticsearch-service-tokens create   elastic/fleet-server fleet-server-token
```

Save the token securely.

---

## 🧩 Step 5: Install Fleet Server (Elastic Agent)

On Fleet Server host:

```bash
sudo ./elastic-agent install   --url=https://192.168.20.128:8220   --fleet-server-es=https://es1:9200   --fleet-server-service-token=<TOKEN>   --fleet-server-policy=fleet-server-policy   --certificate-authorities=/etc/elasticsearch/certs/ca/ca.crt   --fleet-server-es-ca=/etc/elasticsearch/certs/ca/ca.crt   --fleet-server-cert=/etc/elastic-agent/certs/fleet-server.crt   --fleet-server-cert-key=/etc/elastic-agent/certs/fleet-server.key   --fleet-server-port=8220   --install-servers
```

> ✅ Pointing to **one ES node is correct**.  
> Fleet Server is **not data‑path critical**.

---

## 🧩 Step 6: Verify Fleet Server

### Check service
```bash
systemctl status elastic-agent
```

### Check port
```bash
ss -lntp | grep 8220
```

### Kibana UI
Fleet → Agents → Status should be **Healthy**.

---

## 🧪 Common Errors & Fixes

### ❌ x509: unknown authority
✔ Provide CA using:
```
--certificate-authorities
```

### ❌ Fleet Server timeout
✔ Ensure Elasticsearch is **UP**
✔ Ensure Fleet Server can reach ES

### ❌ localhost:9200 confusion
✔ Kibana defaults to localhost  
✔ Fix via Fleet → Settings → Outputs

---

## 🔒 Client Auth (mTLS)

### Client auth = None (Recommended)
- Simpler
- Secure enough for most orgs

### Client auth = Required
- Requires client cert on **every agent**
- High operational overhead

👉 Enable **only if compliance demands**.

---

## 🧠 Best Practices

✔ One Fleet Server per AZ / DC  
✔ Use DNS name in cert SAN  
✔ Monitor Fleet Server via Elastic Agent  
✔ Rotate service tokens periodically  

---

## 🚀 Next Steps

- Enroll regular Elastic Agents
- Add APM integration via Fleet
- Add second Fleet Server for HA
- Decommission legacy Beats

---

## 📌 Summary

Fleet Server is:
- A **control plane**
- Not a data bottleneck
- Safe to connect to **one ES node**
- Secure when TLS + tokens are used correctly

---

Happy Observability 🚀
