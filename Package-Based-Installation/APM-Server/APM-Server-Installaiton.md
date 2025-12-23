# 🚀 Elasticsearch APM Server Deployment Guide (Secure / Production)

This guide explains how to deploy **APM Server** in a **secured Elasticsearch + Kibana cluster** like yours.

---

## 🧠 Architecture Overview

```
Application → APM Agent → APM Server → Elasticsearch → Kibana (APM UI)
```

✔ TLS everywhere  
✔ No `elastic` user for services  
✔ Uses **service account token** (best practice)

---

## 🏗️ Recommended Deployment Model

- ✅ Dedicated APM Server VM (preferred)
- ⚠️ Optional: colocated with Kibana (only for small environments)
- ❌ Do NOT install on Elasticsearch nodes

---

## 🔐 Authentication Strategy

| Component | Method |
|---------|-------|
| APM → Elasticsearch | Service Account Token |
| APM → Kibana | HTTPS + CA trust |
| Apps → APM | Token / agent based |

---

## 📦 Step 1: Install APM Server

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch \
 | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/9.x/apt stable main" \
 | sudo tee /etc/apt/sources.list.d/elastic-9.x.list

sudo chmod 644 /usr/share/keyrings/elasticsearch-keyring.gpg /etc/apt/sources.list.d/elastic-9.x.list

sudo apt-get update
sudo apt install apm-server -y
```

---

## 🔐 Step 2: Trust Elasticsearch CA

```bash
scp es1:/etc/elasticsearch/certs/ca/ca.crt /etc/apm-server/elasticsearch-ca.crt

chown root:apm-server /etc/apm-server/elasticsearch-ca.crt
chmod 640 /etc/apm-server/elasticsearch-ca.crt
```

---

## 🔑 Step 3: Create APM Service Token (on ES node)

```bash
/usr/share/elasticsearch/bin/elasticsearch-service-tokens create elastic/apm apm-server
```

Example output:
```
SERVICE_TOKEN elastic/apm/apm-server = AAEAAWVsYXN0aWMv...
```

---

## ⚙️ Step 4: Configure APM Server

Edit `/etc/apm-server/apm-server.yml`

```yaml
apm-server:
  host: "0.0.0.0:8200"

output.elasticsearch:
  hosts: ["https://es1:9200", "https://es2:9200", "https://es3:9200"]
  service_token: "PASTE_TOKEN_HERE"
  ssl.certificate_authorities: ["/etc/apm-server/elasticsearch-ca.crt"]

setup.kibana:
  host: "https://192.168.20.128:5601"
  ssl.certificate_authorities: ["/etc/apm-server/elasticsearch-ca.crt"]
```

---

## ▶️ Step 5: Enable & Start APM Server

```bash
sudo systemctl daemon-reexec
sudo systemctl enable apm-server
sudo systemctl start apm-server
```

Check status:
```bash
sudo systemctl status apm-server
```

---

## ✅ Step 6: Verification

### From APM Server
```bash
curl -k https://localhost:8200/
```

Expected:
```json
{
  "version": "9.x.x"
}
```

### From Elasticsearch
```bash
curl -u elastic https://es1:9200/_cat/indices/apm*?v
```

### From Kibana
```
Observability → APM → Waiting for data
```

---

## 🧪 Step 7: App Instrumentation (Example)

### Java
```bash
-javaagent:/opt/elastic-apm-agent.jar
-Delastic.apm.server_urls=http://apm-server:8200
-Delastic.apm.service_name=my-app
```

### Node.js
```bash
export ELASTIC_APM_SERVER_URL=http://apm-server:8200
```

---

## 🟢 Best Practices

✔ Use service tokens (no passwords)  
✔ Dedicated APM Server  
✔ TLS everywhere  
✔ Monitor APM with Metricbeat  
✔ Control sampling in production  

---

## 🎯 Next Steps

- Enable APM alerts
- Tune sampling & retention
- Add RUM (browser APM)
- Kubernetes APM

---

✅ Your APM Server is now production-ready.
