# 🛠 Wazuh Hands-on Lab

We’ll do this in **3 parts**:

1. Install Wazuh (all-in-one setup)
2. Add a Linux agent
3. Generate test alerts & view them in the dashboard

---

## 🔹 1. Install Wazuh (All-in-One)

Use a clean VM or server (Ubuntu 22.04 recommended, min 4 GB RAM).

Run the official Wazuh installer:

```bash
curl -sO https://packages.wazuh.com/4.9/wazuh-install.sh
sudo bash wazuh-install.sh -a
```

* `-a` → installs everything:

  * **Wazuh Manager**
  * **Wazuh Indexer (OpenSearch)**
  * **Wazuh Dashboard**

When finished, note the **admin password**.

### Access the dashboard:

👉 `https://<WAZUH_SERVER_IP>:5601`

* Username: `admin`
* Password: shown at the end of the script

---

## 🔹 2. Add a Linux Agent

On the **client machine** you want to monitor (another Ubuntu/Debian VM):

### Install Wazuh agent:

```bash
curl -sO https://packages.wazuh.com/4.9/wazuh-agent-4.9.0.deb
sudo dpkg -i wazuh-agent-4.9.0.deb
```

### Configure agent to connect to server:

Edit `/var/ossec/etc/ossec.conf`:

```xml
<server>
  <address>WAZUH_SERVER_IP</address>
</server>
```

### Start and enable agent:

```bash
sudo systemctl daemon-reexec
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

---

## 🔹 3. Generate Test Alerts

Now let’s make Wazuh produce some **security alerts**.

### (a) Failed SSH login (Brute-force detection)

From another machine, try wrong password logins:

```bash
ssh wronguser@<AGENT_IP>
```

(type wrong password 3–4 times).
👉 Wazuh will detect as **authentication failed / brute force attempt**.

---

### (b) File Integrity Monitoring (FIM)

Modify a critical file on the agent:

```bash
echo "HACKED" | sudo tee -a /etc/passwd
```

👉 Wazuh FIM will trigger an **alert about modified /etc/passwd**.

(⚠️ Don’t leave it broken — remove the line after testing).

---

### (c) Malware Simulation

Create an **EICAR test file** (safe antivirus test string):

```bash
echo "X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*" > /tmp/eicar.com
```

👉 Wazuh will detect this as **malware test file**.

---

## 🔹 4. View Alerts in Dashboard

* Go to Wazuh Dashboard → `Security Events`.
* You should see alerts like:

  * `sshd: authentication failed`
  * `File modified: /etc/passwd`
  * `Malware detected: eicar.com`

You can also query logs in OpenSearch (via Wazuh dashboard search).

---

## 🔹 5. Next Steps

* Add more agents (Windows, Linux, containers).
* Explore **Compliance modules** (PCI DSS, CIS).
* Integrate with **cloud logs** (AWS CloudTrail, Azure Activity Logs).
* Configure **email/Slack alerts** for critical events.

---

✅ With this lab, you’ll have a working Wazuh SIEM system detecting **real security events**.

---
