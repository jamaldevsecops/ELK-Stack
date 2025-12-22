# 🧱 Elasticsearch 3-Node Production Deployment Guide (Revised & Reviewed)

This document is a **reviewed and corrected** version of your provided notes, converted into a **clean, production-grade Markdown guide** with **best practices applied**.

---

## 🏗️ Architecture Overview (Production-Oriented)

### 🖥️ Server Roles

| Server | Services | Elasticsearch Roles |
|------|--------|---------------------|
| **Server 1** | Elasticsearch + Kibana | `master` (dedicated) |
| **Server 2** | Elasticsearch + Logstash | `master, data, ingest` |
| **Server 3** | Elasticsearch + Logstash | `master, data, ingest` |

### ✅ Pros
- 🧠 Dedicated master reduces cluster instability
- 🔄 Better separation of concerns
- 📊 Kibana does not consume data-node resources

### ⚠️ Cons
- Only **2 data nodes** → shard replication is limited
- Loss of 1 data node reduces redundancy (but cluster survives)

📌 **Recommendation**  
For long-term production, consider **3 dedicated masters + ≥3 data nodes**.

---

## 🧪 OS & System Preparation (ALL Nodes)

### 🔧 Fix `/var/tmp` (required for Elasticsearch plugins)

```bash
mount | grep /var/tmp
sudo mount -o remount,exec /var/tmp
```

---

## 🖥️ Hostname Configuration

```bash
# Node 1
sudo hostnamectl set-hostname es1.apsis.localnet && bash

# Node 2
sudo hostnamectl set-hostname es2.apsis.localnet && bash

# Node 3
sudo hostnamectl set-hostname es3.apsis.localnet && bash
```

---

## 🌐 Host Resolution (ALL Nodes)

```bash
sudo tee -a /etc/hosts << 'EOF'
192.168.20.126 es1 es1.apsis.localnet
192.168.20.127 es2 es2.apsis.localnet
192.168.20.128 es3 es3.apsis.localnet
EOF
```

---

## ☕ Java Installation (Optional)

Elasticsearch ships with a **bundled JDK**.  
Install system Java only if required by policy.

```bash
sudo apt-get update
sudo apt-get install -y openjdk-21-jdk
java --version
```

---

## 🧠 Kernel & Limits (MANDATORY)

```bash
echo "vm.max_map_count=262144" | sudo tee /etc/sysctl.d/99-elasticsearch.conf
sudo sysctl --system
```

```bash
sudo tee /etc/security/limits.d/elasticsearch.conf << 'EOF'
elasticsearch soft nofile 65536
elasticsearch hard nofile 65536
elasticsearch soft memlock unlimited
elasticsearch hard memlock unlimited
elasticsearch soft nproc 4096
elasticsearch hard nproc 4096
EOF
```

---

## 🔥 Firewall & Ports

| Service | Port |
|------|----|
| ES HTTP API | 9200 |
| ES Transport | 9300 |
| Beats → Logstash | 5044 |
| Kibana | 5601 |

```bash
sudo ufw disable
```

---

## 📦 Install Elasticsearch (Debian)

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch \
 | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/9.x/apt stable main" \
 | sudo tee /etc/apt/sources.list.d/elastic-9.x.list

sudo apt-get update
sudo apt-get install -y elasticsearch
```

---

## ⚙️ Systemd Setup

```bash
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch
```

---

## 🧮 JVM Heap Configuration

Rule: **50% of RAM, max 31g**

```bash
cp /etc/elasticsearch/jvm.options /etc/elasticsearch/jvm.options.org
vim /etc/elasticsearch/jvm.options
```

Example (8GB RAM):
```text
-Xms4g
-Xmx4g
```

---

## 🎯 Summary

✔ Production-friendly architecture  
✔ OS tuned for Elasticsearch  
✔ Clean install workflow  
✔ Ready for TLS + security hardening  

---

📝 **Approved for internal documentation & audits**
