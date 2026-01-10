# 🧠 Elasticsearch Post-Configuration & Sanity Check Guide

This document should be used **after a successful Elasticsearch cluster bootstrap** (tarball-based installation with TLS enabled).

---

## ✅ Preconditions (Before Running Checks)

Ensure the following are already true:

* Cluster is **GREEN**
* All intended nodes have joined the cluster
* TLS is enabled for **HTTP (9200)** and **Transport (9300)**
* Authentication is working (`elastic` user can log in)

---

## 1️⃣ Reset Built-in `elastic` User Password (FIRST STEP)

After the cluster is up for the first time, **reset the `elastic` superuser password** to ensure security.

### 🔐 Reset Password (Run on ANY node)

```bash
/es/elasticsearch/bin/elasticsearch-reset-password -u elastic
```

* Choose **interactive** mode for a custom password
* Store the password securely (password vault)

### ✅ Verify Login

```bash
curl -u elastic https://es1:9200
```

You should receive a valid JSON response.

---

## 2️⃣ Node Membership & Role Sanity Check

Verify that all nodes have joined correctly.

```bash
curl -u elastic https://es1:9200/_cat/nodes?v
```

Validate:

* All expected nodes are listed
* Node roles are correct (`m`, `dim`, etc.)
* Exactly **one master** is elected (`*`)

---

## 3️⃣ Cluster Health Check

```bash
curl -u elastic https://es1:9200/_cluster/health?pretty
```

### ✅ Expected Output

```json
"status" : "green",
"number_of_nodes" : 3
```

---

## 4️⃣ Remove One-Time Bootstrap Setting (MANDATORY)

`cluster.initial_master_nodes` **must be removed** after the first successful cluster formation.

### 🔧 Action (All Nodes)

Edit:

```bash
/es/elasticsearch/config/elasticsearch.yml
```

Remove this line:

```yaml
cluster.initial_master_nodes: ["es1", "es2", "es3"]
```

---

## 5️⃣ Rolling Restart After Bootstrap Cleanup

Restart nodes **one by one** to apply the change safely.

### 🔁 Recommended Order

1. es1
2. es2
3. es3

Restart command:

```bash
systemctl restart elasticsearch
```

After each restart, confirm the node rejoins the cluster before continuing.

---

## 2️⃣ Cluster Health Verification

```bash
curl -u elastic https://es1:9200/_cluster/health?pretty
```

### ✅ Expected Output

```json
"status" : "green",
"number_of_nodes" : 3
```

---

## 3️⃣ Node Membership & Roles Check

```bash
curl -u elastic https://es1:9200/_cat/nodes?v
```

### Validate:

* All expected nodes are listed
* Node roles are correct (`m`, `dim`, etc.)
* Exactly **one master** is elected (`*`)

---

## 4️⃣ TLS & Security Validation

### HTTP TLS Check

```bash
curl -k -u elastic https://es1:9200
```

Expected:

```json
{"name":"es1","cluster_name":"es-cluster"}
```

### Transport TLS (Node-to-Node)

Check logs on all nodes:

```bash
journalctl -u elasticsearch | grep TLS
```

No handshake or certificate errors should appear.

---

## 5️⃣ Keystore Integrity Check

Run as `elasticsearch` user:

```bash
/es/elasticsearch/bin/elasticsearch-keystore list
```

Ensure required entries exist:

* `xpack.security.http.ssl.keystore.secure_password`
* `xpack.security.transport.ssl.keystore.secure_password`

Permissions:

```bash
ls -l /es/elasticsearch/config/elasticsearch.keystore
```

Expected:

```text
-rw------- elasticsearch elasticsearch
```

---

## 6️⃣ Disk & Shard Allocation Check

```bash
curl -u elastic https://es1:9200/_cat/allocation?v
```

Ensure:

* No shards are unassigned
* Disk usage is within limits

---

## 7️⃣ Persistent Cluster Settings Review

```bash
curl -u elastic https://es1:9200/_cluster/settings?pretty
```

Confirm:

* No leftover bootstrap settings
* No accidental persistent overrides

---

## 8️⃣ Security Hardening (Recommended Next Steps)

* Rotate `elastic` password
* Create least-privilege users and roles
* Disable unused APIs
* Restrict firewall access to 9200

---

## 9️⃣ Backup & Snapshot Sanity

Before production use:

* Configure a snapshot repository (S3 / NFS)
* Take and restore a test snapshot

---

## 🔚 Final Validation Checklist

| Check                      | Status |
| -------------------------- | ------ |
| Cluster GREEN              | ⬜      |
| All nodes joined           | ⬜      |
| TLS verified               | ⬜      |
| Bootstrap config removed   | ⬜      |
| Keystore ownership correct | ⬜      |
| Snapshots tested           | ⬜      |

---

## 🏁 Conclusion

If all checks above pass, your Elasticsearch cluster is:

* ✅ Secure
* ✅ Stable
* ✅ Production-ready

Keep this document for **every new cluster build**.

---

*Document generated for tarball-based Elasticsearch 9.x clusters with TLS enabled.*
