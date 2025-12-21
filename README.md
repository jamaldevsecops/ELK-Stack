# ELK Stack (Elasticsearch, Logstash, Kibana)

## 📌 What is the ELK Stack?
The **ELK Stack** is a popular open-source log management and analytics platform used to **collect, process, store, search, and visualize logs and metrics** from multiple systems in real time.

ELK stands for:
- **E**lasticsearch
- **L**ogstash
- **K**ibana

It is widely used in **DevOps, SRE, Security (SIEM), and Observability** use cases.

---

## 🧱 Components of the ELK Stack

### 🔍 Elasticsearch
Elasticsearch is a **distributed, RESTful search and analytics engine**.

**Key features:**
- Stores data as **JSON documents**
- Near real-time search
- Horizontally scalable
- High availability with replication
- Built on Apache Lucene

**Common use cases:**
- Log storage
- Full-text search
- Metrics analytics
- Security event indexing

**Default ports:**
- `9200` – REST API
- `9300` – Node-to-node communication

---

### 🔄 Logstash
Logstash is a **data processing pipeline** that ingests data from multiple sources, transforms it, and sends it to Elasticsearch or other outputs.

**Capabilities:**
- Supports many input plugins (file, beats, syslog, Kafka, HTTP, etc.)
- Powerful filtering (grok, mutate, date, geoip)
- Multiple output destinations

**Example flow:**
```
Application Logs → Logstash → Elasticsearch
```

---

### 📊 Kibana
Kibana is a **web-based visualization and analytics UI** for Elasticsearch.

**Features:**
- Dashboards and charts
- Log exploration (Discover)
- Alerting and monitoring
- Role-based access control
- Elasticsearch management UI

**Default port:**
- `5601`

---

## 🔁 ELK Data Flow Architecture

```
[ Applications / Servers ]
            |
            v
      [ Logstash ]
            |
            v
     [ Elasticsearch ]
            |
            v
        [ Kibana ]
```

Optionally, **Beats** (like Filebeat, Metricbeat) are used to ship data efficiently to Logstash or directly to Elasticsearch.

---

## 🚀 Why Use ELK Stack?

✅ Centralized logging  
✅ Fast search and analytics  
✅ Scales horizontally  
✅ Open-source and extensible  
✅ Strong ecosystem and community  

---

## 🔐 Security in ELK
Modern ELK (Elastic Stack) includes:
- TLS encryption (HTTP & transport layer)
- User authentication & RBAC
- Audit logging
- API key support

---

## 🛠 Common Use Cases
- Application & system log monitoring
- Infrastructure observability
- Security monitoring (SIEM)
- Troubleshooting & root cause analysis
- Compliance and auditing

---

## 📦 ELK vs Elastic Stack
The term **Elastic Stack** extends ELK by adding:
- **Beats**
- **Elastic Agent**
- **Fleet**
- **APM**

---

## 📝 Summary
The ELK Stack is a powerful and flexible platform for log management and analytics. It is a core tool in modern DevOps environments and integrates well with cloud, containerized, and on-prem infrastructures.

---

## 📚 References
- Elasticsearch Documentation
- Logstash Documentation
- Kibana Documentation
