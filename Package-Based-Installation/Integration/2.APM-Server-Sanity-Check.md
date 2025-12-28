# 🔍 APM Server Sanity Check Guide (Secure Elasticsearch Setup)

This document helps you **verify, validate, and troubleshoot** an Elastic APM Server deployment
integrated with a **secured Elasticsearch cluster and Kibana**, based on your **current working setup**.

---

## 🧩 Architecture Overview

```
[ Application ]
      |
      |  (APM data via HTTP 8200)
      v
[ APM Server ]
      |
      |  (TLS + apm_system user)
      v
[ Elasticsearch Cluster ]
      |
      v
[ Kibana UI ]
```

---

## ✅ 1. APM Server Service Health

### 🔹 Check systemd status
```bash
systemctl status apm-server
```

Expected:
```
Active: active (running)
```

If failed → check logs:
```bash
journalctl -u apm-server -n 100 --no-pager
```

---

## ✅ 2. Configuration Validation

### 🔹 Validate configuration syntax
```bash
apm-server test config -e
```

Expected:
```
Config OK
```

> ⚠️ `apm-server test output` may panic on Elastic 9.x due to a known upstream bug.
> This does **NOT** indicate runtime failure.

---

## ✅ 3. APM Intake Endpoint Check

APM Server does not expose a human-readable `/` endpoint.

### 🔹 Validate intake endpoint is alive
```bash
curl -X POST http://192.168.20.129:8200/intake/v2/events
```

Expected:
```json
{"error":"EOF"}
```

This confirms:
- APM Server is listening
- Intake pipeline is active

---

## ✅ 4. Elasticsearch Connectivity (APM Server → ES)

APM Server authenticates using the **built-in `apm_system` user**.

### 🔹 Confirm Elasticsearch cluster is reachable
```bash
curl -k -u elastic https://es1:9200/_cluster/health?pretty
```

Expected:
```json
"status" : "green"
```

---

## ✅ 5. APM Indices Check (After Agent Data)

APM indices are created **only after the first agent sends data**.

### 🔹 Check APM indices
```bash
curl -k -u elastic https://es1:9200/_cat/indices/apm*?v
```

Before agents:
```
(no indices shown)
```

After agents:
```
apm-9.x.x-transaction
apm-9.x.x-metric
apm-9.x.x-span
apm-9.x.x-error
```

---

## ✅ 6. Kibana Integration Check

### 🔹 Verify Kibana is reachable
```bash
curl -k https://192.168.20.128:5601/api/status
```

Expected:
```json
"overall":{"state":"green"}
```

### 🔹 UI Verification
Navigate to:
```
Kibana → Observability → APM
```

Expected:
- No errors
- “No services detected yet” (until agents connect)

---

## ✅ 7. Authentication Model Validation

| Component | Auth Method |
|--------|-----------|
| APM Server → Elasticsearch | `apm_system` user |
| APM Agents → APM Server | API Key |
| Kibana → Elasticsearch | `kibana` system user |
| Admin Access | `elastic` user |

✔ Correct production-grade model

---

## ✅ 8. Common Warnings (Safe to Ignore)

### 🔸 `beater.agentcfg` errors
Cause:
- No APM agent configuration exists yet in Kibana

Resolution:
- Automatically disappears after agents connect

---

## 🚫 What NOT to Expect

| Check | Expected |
|----|----|
`curl http://apm-server:8200/` | Empty / no output |
APM indices before agents | ❌ |
Successful `test output` on 9.x | ❌ |

---

## 🧪 9. Final Readiness Checklist

- [x] APM Server running
- [x] TLS enforced
- [x] Correct authentication
- [x] Kibana reachable
- [x] Ready for agent onboarding

🎉 **Your APM Server is production-ready.**

---
Next: []()  
Prev: []() 
