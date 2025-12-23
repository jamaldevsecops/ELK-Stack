# 🔗 Kibana Configuration & Elasticsearch Integration Guide

This document explains how to **configure Kibana**, **integrate it securely with Elasticsearch**, and follow **best practices for users**.

---

## 🔐 Background (Important)

During Elasticsearch installation, security auto-configuration enabled:

- Authentication & authorization
- TLS for HTTP & transport
- Generated `elastic` superuser password

⚠️ `elastic` is for **initial setup only**.

---

## 👤 User Strategy (Best Practice)

| User | Purpose |
|----|----|
| `elastic` | Emergency / initial admin |
| `kibana` | Kibana backend service user |
| `kibana_admin` | Human admin login |
| others | Dashboard / read-only users |

🚫 Do NOT log in to Kibana UI with `kibana`.

---

## 🧩 1. Create Dedicated Kibana Admin User

```bash
curl -u elastic -X PUT https://es1:9200/_security/role/kibana_admin_custom -H "Content-Type: application/json" -d '
{
  "cluster": ["monitor"],
  "indices": [
    { "names": ["*"], "privileges": ["all"] }
  ],
  "applications": [
    {
      "application": "kibana-.kibana",
      "privileges": ["all"],
      "resources": ["*"]
    }
  ]
}'
```

```bash
curl -u elastic -X POST https://es1:9200/_security/user/kibana_admin -H "Content-Type: application/json" -d '
{
  "password": "STRONG_PASSWORD",
  "roles": ["kibana_admin_custom"],
  "full_name": "Kibana Admin User"
}'
```

---

## ⚙️ 2. Configure `kibana.yml`

Create Kibana Encryption Keys: 
```
openssl rand -base64 48
```

```yaml
# =================== System: Kibana Server ===================
server.port: 5601
server.host: "192.168.20.128"

#server.basePath: ""
#server.rewriteBasePath: false
server.publicBaseUrl: "http://192.168.20.128:5601"
#server.maxPayload: 1048576
server.name: "NGD-Kibana"

# =================== System: Kibana Server (Optional) ===================
#server.ssl.enabled: false
#server.ssl.certificate: /path/to/your/server.crt
#server.ssl.key: /path/to/your/server.key

# =================== System: Elasticsearch ===================
elasticsearch.hosts: ["https://es1:9200", "https://es2:9200", "https://es3:9200"]
elasticsearch.username: "kibana"
elasticsearch.password: "H4PYrdDPLo18HttdJ1z9"

# Kibana can also authenticate to Elasticsearch via "service account tokens".
# Service account tokens are Bearer style tokens that replace the traditional username/password based configuration.
# Use this token instead of a username/password.
# elasticsearch.serviceAccountToken: "my_token"

#elasticsearch.pingTimeout: 1500
#elasticsearch.requestTimeout: 30000
#elasticsearch.maxSockets: 1024
#elasticsearch.compression: false

#elasticsearch.requestHeadersWhitelist: [ authorization ]
#elasticsearch.customHeaders: {}
#elasticsearch.shardTimeout: 30000

# =================== System: Elasticsearch (Optional) ===================
#elasticsearch.ssl.certificate: /path/to/your/client.crt
#elasticsearch.ssl.key: /path/to/your/client.key
elasticsearch.ssl.certificateAuthorities: [ "/etc/kibana/certs/elasticsearch-ca.crt" ]
elasticsearch.ssl.verificationMode: full

# =================== System: Logging ===================
# Set the value of this setting to off to suppress all logging output, or to debug to log everything. Defaults to 'info'
#logging.root.level: debug

# Enables you to specify a file where Kibana stores log output.
logging:
  appenders:
    file:
      type: file
      fileName: /var/log/kibana/kibana.log
      layout:
        type: json
  root:
    appenders:
      - default
      - file
#  policy:
#    type: size-limit
#    size: 256mb
#  strategy:
#    type: numeric
#    max: 10
#  layout:
#    type: json

# Logs queries sent to Elasticsearch.
#logging.loggers:
#  - name: elasticsearch.query
#    level: debug

# Logs http responses.
#logging.loggers:
#  - name: http.server.response
#    level: debug

# Logs system usage information.
#logging.loggers:
#  - name: metrics.ops
#    level: debug

# Enables debug logging on the browser (dev console)
#logging.browser.root:
#  level: debug

# =================== System: Other ===================
#path.data: data
pid.file: /run/kibana/kibana.pid
#ops.interval: 5000
#i18n.locale: "en"

# =================== Frequently used (Optional)===================
# Kibana internal encryption keys (REQUIRED)
xpack.security.encryptionKey: "lDfeb9eqBh1/40ao+/3mbMrTS2rBArf5zhxXDvQSrGJuQNCl91fgcFNZatUvMlPA"
xpack.encryptedSavedObjects.encryptionKey: "EDts25VGQJbcYc+5EsMz7OlfJ0A3Qu0v4o1Tjm4zF+U5jAzmSOSXQFClawSWNIa1"
xpack.reporting.encryptionKey: "NyuH/OBP7ABpyLL3dfhp29SEKBhTeE9yam8E6O0Rt4ljmxZoEfc5w4J4mQ49g7Iz"
# =================== Saved Objects: Migrations ===================
#migrations.batchSize: 1000
#migrations.maxBatchSizeBytes: 100mb
#migrations.retryAttempts: 15

# =================== Search Autocomplete ===================
#unifiedSearch.autocomplete.valueSuggestions.timeout: 1000
#unifiedSearch.autocomplete.valueSuggestions.terminateAfter: 100000
```

---

## 🔑 3. Restart Kibana

```bash
sudo systemctl restart kibana
journalctl -u kibana -f
```

Expected:
```
Kibana is now available at http://<host>:5601
```
<img width="1646" height="689" alt="image" src="https://github.com/user-attachments/assets/21af4cf6-cf29-42a4-8477-86eded0ab6e9" />

---

## 🌐 4. Log in to Kibana

🔑 **Admin login**
```
Username: kibana_admin
Password: STRONG_PASSWORD
```

🛑 Do NOT use:
```
kibana / <password>
```

---

## 🟢 Verification Checklist

✔ Kibana UI loads  
✔ No TLS warnings  
✔ Saved objects persist after restart  
✔ Elasticsearch cluster visible  

---
Next: []()  
Prev: []()  

