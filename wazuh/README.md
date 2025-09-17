# 📘 Wazuh – Detailed Notes

## 🔹 What is Wazuh?

* **Wazuh** is an open-source **SIEM (Security Information and Event Management)** and **XDR (Extended Detection & Response)** platform.
* Originally a fork of **OSSEC**, now a full security platform.
* Free and open-source.

---

## 🔹 Key Features

1. **Log Collection & Analysis**

   * Collects logs from Linux, Windows, MacOS, Cloud (AWS, Azure, GCP), containers, firewalls, etc.
   * Normalizes and indexes logs for searching.

2. **Threat Detection & Intrusion Detection**

   * Detects brute-force, malware, privilege escalation attempts, rootkits.
   * Uses **rules and decoders** (similar to Splunk’s SPL queries but event-driven).

3. **File Integrity Monitoring (FIM)**

   * Monitors changes in system files, configs, binaries.

4. **Vulnerability Detection**

   * Integrates with vulnerability databases (CVE, NVD) to identify risks.

5. **Compliance Monitoring**

   * Built-in checks for standards like **PCI DSS, HIPAA, GDPR, NIST, CIS**.

6. **SIEM & Correlation**

   * Collects events, correlates them, and generates alerts.
   * Dashboards for security teams.

7. **Cloud & Container Security**

   * AWS CloudTrail, Azure, GCP logging integration.
   * Docker & Kubernetes monitoring.

---

## 🔹 Wazuh Architecture

Wazuh is typically deployed in **three main components**:

1. **Wazuh Agent**

   * Installed on endpoints (Linux, Windows, macOS, containers).
   * Collects logs, monitors files, runs rootkit detection.

2. **Wazuh Server / Manager**

   * Central brain:

     * Decodes and analyzes data from agents.
     * Runs correlation rules.
     * Stores alerts.

3. **Indexing + Dashboard**

   * Uses **OpenSearch** (or Elasticsearch) to index logs.
   * Uses **Wazuh Dashboard** (Kibana plugin or web UI) to visualize and query.

👉 So it’s very similar to Splunk’s architecture:

* Agents (like Splunk Forwarders) → Manager (like Splunk Indexer) → OpenSearch (like Splunk Index) → Dashboard.

---

# 🛠 Getting Started with Wazuh

## 🔹 1. System Requirements

* **OS**: Ubuntu 20.04/22.04, CentOS, Debian (Linux recommended).
* **RAM**: At least 4 GB (8 GB+ for production).
* **Disk**: 20 GB minimum.
* **Dependencies**: OpenSearch/Elasticsearch, Node.js (handled by Wazuh installer).

---

## 🔹 2. Install Wazuh (All-in-One Deployment)

Wazuh provides a quick-install script for lab use. On a **clean Ubuntu 22.04 server**:

```bash
curl -sO https://packages.wazuh.com/4.9/wazuh-install.sh
sudo bash wazuh-install.sh -a
```

* `-a` → All-in-one installation (manager + dashboard + indexer).
* This installs:

  * **Wazuh Manager**
  * **Wazuh Indexer (OpenSearch fork)**
  * **Wazuh Dashboard**

After install, access dashboard at:
👉 `https://<server-ip>:5601`
Default credentials:

* **Username**: `admin`
* **Password**: shown at end of installation.

---

## 🔹 3. Add an Agent

Install agent on a client (e.g., Ubuntu):

```bash
curl -sO https://packages.wazuh.com/4.9/wazuh-agent-4.9.0.deb
sudo dpkg -i wazuh-agent-4.9.0.deb
```

Configure agent to point to Wazuh Manager:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Set:

```xml
<server>
  <address>WAZUH-MANAGER-IP</address>
</server>
```

Start agent:

```bash
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

---

## 🔹 4. Verify & Explore

* In **Wazuh Dashboard**, check:

  * `Agents` → See connected hosts.
  * `Security Events` → Intrusion detection alerts.
  * `Integrity Monitoring` → File change events.
  * `Vulnerabilities` → Detected CVEs.

---

## 🔹 5. Example Use Cases

* **Detect failed SSH logins**:

  * Wazuh parses `/var/log/auth.log` → Detects brute-force attempts.
* **Monitor critical file changes**:

  * If `/etc/passwd` or `/etc/ssh/sshd_config` changes → alert.
* **Cloud security**:

  * Collect AWS CloudTrail logs → detect IAM privilege escalations.

---

## 🔹 6. Example Alert JSON

Wazuh stores alerts in JSON (queryable in OpenSearch). Example failed SSH login:

```json
{
  "rule": {
    "id": "5710",
    "level": 5,
    "description": "sshd: authentication failed."
  },
  "agent": {
    "id": "002",
    "name": "ubuntu-client"
  },
  "data": {
    "srcip": "192.168.1.25",
    "user": "root"
  },
  "timestamp": "2025-09-17T06:30:15.000Z"
}
```

---

## 🔹 7. Wazuh vs Splunk

| Feature            | Wazuh (Open Source) | Splunk (Commercial)               |
| ------------------ | ------------------- | --------------------------------- |
| Licensing          | Free                | Expensive (GB/day pricing)        |
| Log Collection     | Agents, syslog      | Universal Forwarder               |
| Analytics          | Rules, OpenSearch   | SPL (advanced)                    |
| Security Use Cases | Strong (IDS, SIEM)  | Strong (with Enterprise Security) |
| Ecosystem          | Smaller             | Very large                        |

---

✅ **Summary:**

* Wazuh is **best when you need SIEM, compliance, threat detection** with zero licensing cost.
* For general **log analytics** → OpenSearch / ELK might be better.
* For **observability in K8s** → Loki + Grafana is lighter.

---


