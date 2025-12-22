# 🔐 Elasticsearch Production TLS Certificate Guide (Per-Node, Password-Protected)

This guide explains **from scratch** how to generate **production‑grade TLS certificates** for a **multi‑node Elasticsearch cluster**, using **one CA** and **per‑node certificates** (best practice).

---

## 📌 Architecture Overview

```
          🏛️  Root CA (ca.crt)
                 │
   ┌─────────────┼─────────────┐
   │             │             │
🔐 es1.p12     🔐 es2.p12     🔐 es3.p12
   │             │             │
  ES Node 1     ES Node 2     ES Node 3
```

- 🔑 One **Certificate Authority (CA)**
- 🔐 One **PKCS#12 certificate per node**
- 🔒 Passwords stored in `elasticsearch.keystore`
- ❌ No auto‑enrollment (manual PKI)

---

## 🧱 Prerequisites

- Elasticsearch 8.x / 9.x installed
- Nodes:
  - `es1` → `192.168.20.128`
  - `es2` → `192.168.20.129`
  - `es3` → `192.168.20.130`
- OpenSSL installed
- Root access

---

## 🏛️ Step 1 — Create a Production Certificate Authority (CA)

📍 Run **only on es1**

```bash
mkdir -p /etc/elasticsearch/certs/ca
cd /etc/elasticsearch/certs/ca

# 🔑 Generate CA private key (KEEP SAFE!)
openssl genrsa -out ca.key 4096
chmod 600 ca.key

# 📜 Generate CA certificate
openssl req -x509 -new -key ca.key -sha512 -days 3650 -out ca.crt \
  -subj "/C=BD/ST=Dhaka/L=Dhaka/O=Nagad/OU=IT/CN=nagad-es-ca"
```

✅ Results:
- `ca.key` → 🔒 **Private (do NOT copy)**
- `ca.crt` → 📢 Public (copy to all nodes)

---

## 🧾 Step 2 — Generate Per‑Node Certificates (Best Practice)

### 📁 Directory layout
```bash
mkdir -p /etc/elasticsearch/certs/nodes/es1
cd /etc/elasticsearch/certs/nodes/es1
```

---

### 🧩 OpenSSL config (es1)

Create `es1-openssl.cnf`:

```ini
[ req ]
distinguished_name = req_distinguished_name
req_extensions     = v3_req
prompt             = no

[ req_distinguished_name ]
CN = es1

[ v3_req ]
keyUsage = digitalSignature, keyAgreement
extendedKeyUsage = serverAuth, clientAuth
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = es1
DNS.2 = es1.apsis.localnet
IP.1  = 192.168.20.128
```

---

### 🔑 Generate key + CSR

```bash
openssl genrsa -out es1.key 4096
openssl req -new -key es1.key -out es1.csr -config es1-openssl.cnf
```

---

### 🖊️ Sign CSR with CA

```bash
openssl x509 -req -in es1.csr \
  -CA ../../ca/ca.crt \
  -CAkey ../../ca/ca.key \
  -CAcreateserial \
  -out es1.crt \
  -days 825 -sha512 \
  -extensions v3_req \
  -extfile es1-openssl.cnf
```

---

### 🔐 Create password‑protected PKCS#12

```bash
openssl pkcs12 -export \
  -in es1.crt \
  -inkey es1.key \
  -certfile ../../ca/ca.crt \
  -out es1.p12
```

👉 Enter a **strong password**  
👉 Use the **same password for all nodes** (recommended)

---

### 🔁 Repeat for es2 and es3

Only change:
- `CN`
- `DNS`
- `IP`

Directories:
```
/etc/elasticsearch/certs/nodes/es2
/etc/elasticsearch/certs/nodes/es3
```

---

## 📦 Step 3 — Copy Certificates to Other Nodes

```bash
# es2
scp -r /etc/elasticsearch/certs/nodes/es2 es2:/etc/elasticsearch/certs/nodes/
scp /etc/elasticsearch/certs/ca/ca.crt es2:/etc/elasticsearch/certs/ca/

# es3
scp -r /etc/elasticsearch/certs/nodes/es3 es3:/etc/elasticsearch/certs/nodes/
scp /etc/elasticsearch/certs/ca/ca.crt es3:/etc/elasticsearch/certs/ca/
```

🚫 Never copy `ca.key`

---

## 🔒 Step 4 — Fix Permissions (ALL nodes)

```bash
chown -R root:elasticsearch /etc/elasticsearch/certs
find /etc/elasticsearch/certs -type f -exec chmod 660 {} \;
find /etc/elasticsearch/certs -type d -exec chmod 755 {} \;
```

---

## 🔑 Step 5 — Configure elasticsearch.keystore (ALL nodes)

Each node must store **its own `.p12` password**.

```bash
/usr/share/elasticsearch/bin/elasticsearch-keystore create

/usr/share/elasticsearch/bin/elasticsearch-keystore add \
  xpack.security.http.ssl.keystore.secure_password

/usr/share/elasticsearch/bin/elasticsearch-keystore add \
  xpack.security.transport.ssl.keystore.secure_password
```

✅ Verify:
```bash
/usr/share/elasticsearch/bin/elasticsearch-keystore list
```

---

## ⚙️ Step 6 — Elasticsearch TLS Configuration (es1 example)

```yaml
xpack.security.enabled: true
xpack.security.enrollment.enabled: false

xpack.security.http.ssl:
  enabled: true
  keystore.path: /etc/elasticsearch/certs/nodes/es1/es1.p12

xpack.security.transport.ssl:
  enabled: true
  verification_mode: certificate
  keystore.path: /etc/elasticsearch/certs/nodes/es1/es1.p12
  certificate_authorities:
    - /etc/elasticsearch/certs/ca/ca.crt
```

📌 Use `certificate_authorities` for `.crt` (PEM)  
📌 Use `truststore.path` only for `.p12` / `.jks`

---

## ▶️ Step 7 — Start the Cluster

```bash
systemctl start elasticsearch
```

Start order:
1️⃣ es1  
2️⃣ es2  
3️⃣ es3  

---

## 🧪 Verification

```bash
curl -k -u elastic https://es1:9200/_cluster/health?pretty
```

Expected:
```json
"status": "green",
"number_of_nodes": 3
```

---

## 🧠 Key Rules to Remember

| Item | Rule |
|----|----|
| 🔐 Node cert | `.p12` |
| 🏛️ CA cert | `.crt` (PEM) |
| 🔑 Passwords | `elasticsearch.keystore` |
| ❌ Enrollment | Disable for manual PKI |
| 🔁 Rotation | Replace cert → restart node |

---

## ✅ Outcome

You now have:

✔ Production‑grade CA  
✔ Per‑node TLS identity  
✔ Password‑protected PKCS#12  
✔ Secure keystore handling  
✔ Fully encrypted ES cluster  

---

🎉 **Congratulations — this is enterprise‑level Elasticsearch security.**
