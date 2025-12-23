# 🚀 Elasticsearch APM Server Deployment Guide (Elastic 9.x – Secure / Production)

This guide describes how to deploy **APM Server** in an **Elastic 9.x secured cluster** using the **supported authentication model**.

---

## 🧠 Architecture Overview

```
Application → APM Agent → APM Server → Elasticsearch → Kibana (APM UI)
```

✔ TLS on all layers  
✔ Enrollment disabled (production best practice)  
✔ Uses **apm_system** built-in user (Elastic 9.x supported)

---

## 🏗️ Recommended Deployment Model

- ✅ Dedicated APM Server VM (recommended)
- ⚠️ Can be colocated with Kibana for small setups 
- ❌ Do NOT deploy APM Server on Elasticsearch nodes (adds ingest pressure)

---

## 🔐 Authentication Model (Important)

In **Elastic 9.x**, APM Server:

- ❌ Does NOT support service account tokens
- ✅ Uses the built-in **`apm_system`** user
- ❌ Must NOT use `elastic` user

| Component | Auth Method |
|--------|------------|
| APM → Elasticsearch | `apm_system` user |
| APM → Kibana | HTTPS + CA trust |
| Apps → APM | Agent-based |

---

## 📦 Step 1: Install APM Server

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch \
 | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/9.x/apt stable main" \
 | sudo tee /etc/apt/sources.list.d/elastic-9.x.list

sudo chmod 644 /usr/share/keyrings/elasticsearch-keyring.gpg /etc/apt/sources.list.d/elastic-9.x.list

sudo apt-get update
sudo apt list apm-server -a 
sudo apt install apm-server=9.2.3 -y
```

---

## 🔐 Step 2: Trust Elasticsearch CA

```bash
scp es1:/etc/elasticsearch/certs/ca/ca.crt /etc/apm-server/elasticsearch-ca.crt

chown root:apm-server /etc/apm-server/elasticsearch-ca.crt
chmod 640 /etc/apm-server/elasticsearch-ca.crt
```

---

## 🔑 Step 3: Reset `apm_system` Password (on ES node)

```bash
/usr/share/elasticsearch/bin/elasticsearch-reset-password -u apm_system
```

Example:
```
Password for the [apm_system] user successfully reset.
New value: ********
```

---

## ⚙️ Step 4: Configure APM Server

Edit `/etc/apm-server/apm-server.yml`

```yaml
apm-server:
  host: "0.0.0.0:8200"

output.elasticsearch:
  hosts:
    - "https://es1:9200"
    - "https://es2:9200"
    - "https://es3:9200"
  username: "apm_system"
  password: "PASTE_PASSWORD_HERE"
  ssl.certificate_authorities:
    - "/etc/apm-server/elasticsearch-ca.crt"

setup.kibana:
  host: "https://192.168.20.128:5601"
  ssl.certificate_authorities:
    - "/etc/apm-server/elasticsearch-ca.crt"
```

---

## 🔐 File Permissions

```bash
chown root:apm-server /etc/apm-server/apm-server.yml
chmod 640 /etc/apm-server/apm-server.yml
```

---

## ▶️ Step 5: Enable & Start APM Server

```bash
systemctl daemon-reexec
systemctl enable apm-server
systemctl restart apm-server
```

---

## ✅ Step 6: Verification

```bash
curl -k https://localhost:8200/
curl -u elastic https://es1:9200/_cat/indices/apm*?v
```

Kibana:
```
Observability → APM → Waiting for data
```

---

## 🧪 Step 7: Instrument Applications (Example)

### Java
```bash
-javaagent:/opt/elastic-apm-agent.jar
-Delastic.apm.server_urls=http://apm-server:8200
```

### Node.js
```bash
export ELASTIC_APM_SERVER_URL=http://apm-server:8200
```

---

## 🟢 Best Practices

✔ Enrollment disabled  
✔ apm_system user only  
✔ TLS everywhere  
✔ Dedicated APM Server  

---

## 🎯 Next Steps

- Enable alerts
- Tune sampling
- Add RUM
- Kubernetes APM

---
Nex: []()  
Prev: []()  
