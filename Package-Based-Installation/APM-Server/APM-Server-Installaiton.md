# 🚀 Elasticsearch APM Server Deployment Guide (Elastic 9.x – Secure / Production)

This guide describes how to deploy **APM Server** in an **Elastic 9.x secured cluster** using the **supported authentication model** and **HTTPS-enabled APM intake**.

---

## 🧠 Architecture Overview

```
Application → APM Agent → APM Server (HTTPS :8200) → Elasticsearch → Kibana (APM UI)
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

| Component | Authentication |
|--------|---------------|
| APM → Elasticsearch | `apm_system` user |
| APM → Kibana | HTTPS + CA trust |
| Applications → APM | API Key / Secret Token |

---

## 📦 Step 1: Install APM Server

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch  | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/9.x/apt stable main"  | sudo tee /etc/apt/sources.list.d/elastic-9.x.list

sudo chmod 644 /usr/share/keyrings/elasticsearch-keyring.gpg /etc/apt/sources.list.d/elastic-9.x.list

sudo apt-get update
sudo apt install apm-server=9.2.3 -y
```

---

## 🔐 Step 2: Trust Elasticsearch CA

```bash
mkdir -p /etc/apm-server/certs/ca
scp root@es1:/etc/elasticsearch/certs/ca/ca.crt /etc/apm-server/certs/ca/ca.crt

chown apm-server:apm-server /etc/apm-server/certs/ca/ca.crt
chmod 640 /etc/apm-server/certs/ca/ca.crt
```

---

## 🔐 Step 3: Generate HTTPS Certificate for APM Server

```bash
mkdir -p /etc/apm-server/certs/apm
cd /etc/apm-server/certs/apm
```

### Create OpenSSL config
```bash
cat > apm-openssl.cnf <<EOF
[ req ]
prompt = no
distinguished_name = dn
req_extensions = v3_req

[ dn ]
CN = apm-server

[ v3_req ]
keyUsage = digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = apm-server
DNS.2 = es2
DNS.3 = es2.apsis.localnet
IP.1  = 192.168.20.129
EOF
```

### Generate key and certificate
```bash
openssl genrsa -out apm-server.key 4096

openssl req -new -key apm-server.key   -out apm-server.csr   -config apm-openssl.cnf

openssl x509 -req -in apm-server.csr   -CA /etc/apm-server/certs/ca/ca.crt   -CAkey /etc/apm-server/certs/ca/ca.key   -CAcreateserial   -out apm-server.crt   -days 825 -sha256   -extensions v3_req   -extfile apm-openssl.cnf
```

```bash
chown apm-server:apm-server apm-server.*
chmod 600 apm-server.key
chmod 644 apm-server.crt
```

---

## 🔑 Step 4: Reset `apm_system` Password (on ES node)

```bash
/usr/share/elasticsearch/bin/elasticsearch-reset-password -u apm_system
```

---

## ⚙️ Step 5: Configure APM Server

Edit `/etc/apm-server/apm-server.yml`

```yaml
apm-server:
  host: "192.168.20.129:8200"
  ssl:
    enabled: true
    certificate: "/etc/apm-server/certs/apm/apm-server.crt"
    key: "/etc/apm-server/certs/apm/apm-server.key"
  auth:
    api_key:
      enabled: true
      limit: 100

output.elasticsearch:
  hosts: ["https://es1:9200", "https://es2:9200", "https://es3:9200"]
  username: "apm_system"
  password: "<APM_SYSTEM_PASSWORD>"
  ssl:
    enabled: true
    verification_mode: full
    certificate_authorities: ["/etc/apm-server/certs/ca/ca.crt"]

kibana:
  enabled: true
  host: "https://192.168.20.128:5601"
  ssl:
    enabled: true
    verification_mode: full
    certificate_authorities:
      - "/etc/apm-server/certs/ca/ca.crt"
```

---

## 🔐 File Permissions

```bash
chown root:apm-server /etc/apm-server/apm-server.yml
chmod 640 /etc/apm-server/apm-server.yml
```

---

## ▶️ Step 6: Enable & Start APM Server

```bash
systemctl daemon-reexec
systemctl enable apm-server
systemctl restart apm-server
systemctl status apm-server
```

---

## ✅ Verification

```bash
curl --cacert /etc/apm-server/certs/ca/ca.crt https://192.168.20.129:8200/
```

✔ HTTPS reachable  
✔ No certificate errors

---

**Next:** ILM tuning for APM indices  
**Prev:** Kibana integration
