# 🔐 Intrusion Detection Project with Wazuh

## 📋 Description

This project sets up an intrusion detection and security monitoring infrastructure based on **Wazuh**. The environment consists of three machines: a Wazuh server, an Ubuntu server hosting containerized vulnerable applications, and a Kali Linux machine used to simulate attacks.

The goal is to demonstrate Wazuh's ability to detect, alert, and log various security events (web attacks, file modifications, known vulnerabilities).

---

## 🏗️ Project Architecture

```
┌─────────────────────┐        ┌─────────────────────┐
│   Kali Machine      │        │   Wazuh Machine     │
│   (Attacker)        │───────▶│   (SIEM Server)     │
│   IP: 192.168.x.x   │        │   IP: 192.168.x.x   │
└─────────────────────┘        └──────────┬──────────┘
                                           │
                                           │ Local Network
                                           │
                                ┌──────────▼──────────┐
                                │   Ubuntu Machine    │
                                │   (Target / Agent)  │
                                │   IP: 192.168.x.x   │
                                │                     │
                                │  ┌───────────────┐  │
                                │  │ Docker        │  │
                                │  │ ├─ DVWA       │  │
                                │  │ └─ Juice Shop │  │
                                │  └───────────────┘  │
                                └─────────────────────┘
```

---

## 🖥️ Infrastructure Components

| Machine  | Role | Services |
|----------|------|----------|
| **Wazuh** | SIEM / Manager Server | Wazuh Manager, Indexer, Dashboard |
| **Ubuntu** | Monitored Target | Wazuh Agent, Docker, DVWA, Juice Shop |
| **Kali** | Attacker | Pentest tools (nmap, sqlmap, hydra, etc.) |

---

## ⚙️ Installation and Configuration

### 1. Wazuh Machine (Server)

- Installation of the Wazuh server (Manager, Indexer, Dashboard)
- Network configuration to allow communication with the Ubuntu agent
- Access to the Wazuh dashboard via `https://<WAZUH_IP>`

### 2. Ubuntu Machine (Agent)

#### a) Network Configuration
- IP address configuration and connectivity with the Wazuh server
- Communication verification via `ping` and `netstat`

#### b) Wazuh Agent Installation
```bash
# Add the Wazuh repository
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --dearmor > /usr/share/keyrings/wazuh.gpg

# Install the agent
apt-get install wazuh-agent

# Configure the agent (point to the manager)
# /var/ossec/etc/ossec.conf → <address>WAZUH_IP</address>

# Start the agent
systemctl enable wazuh-agent
systemctl start wazuh-agent
```

#### c) Docker Installation
```bash
sudo apt update
sudo apt install docker.io docker-compose -y
sudo systemctl enable --now docker
```

#### d) Deploying DVWA and Juice Shop
```bash
# DVWA
docker run -d -p 8080:80 --name dvwa vulnerables/web-dvwa

# OWASP Juice Shop
docker run -d -p 3000:3000 --name juice-shop bkimminich/juice-shop
```

- **DVWA**: `http://<UBUNTU_IP>:8080`
- **Juice Shop**: `http://<UBUNTU_IP>:3000`

---

## 🧪 Testing and Validation

### ✅ 1. Verifying Application Alerts (DVWA & Juice Shop)

- Accessing the web applications and performing normal / abnormal requests
- Checking the **Wazuh Dashboard** for log collection (via Docker / syslog logs)

### ✅ 2. Attack Simulation from Kali

Tools used to generate malicious traffic:

```bash
# Port scanning
nmap -sS -A <UBUNTU_IP>

# Brute force attack on DVWA (Hydra)
hydra -l admin -P /usr/share/wordlists/rockyou.txt <UBUNTU_IP> http-post-form "/login.php:username=^USER^&password=^PASS^&Login=Login:Login failed"

# SQL injection on DVWA / Juice Shop
sqlmap -u "http://<UBUNTU_IP>:8080/vulnerabilities/sqli/?id=1&Submit=Submit" --cookie="..."
```

**Expected result**: Alerts raised in the Wazuh dashboard (web detection rules, brute force, etc.)

### ✅ 3. Vulnerability Detection

- Enabling the module in the agent configuration:
  ```xml
  <syscollector>
    <disabled>no</disabled>
    <interval>3600</interval>
  </syscollector>
  
  <vulnerability-detector>
    <enabled>yes</enabled>
    <interval>5m</interval>
  </vulnerability-detector>
  ```
- Restarting the agent:
  ```bash
  systemctl restart wazuh-agent
  ```
- Checking the **Dashboard → Vulnerability Detection** to view CVEs detected on Ubuntu (packages, libraries, Docker, etc.)

### ✅ 4. File Integrity Monitoring (FIM)

- Configuration in `ossec.conf`:
  ```xml
  <syscheck>
    <disabled>no</disabled>
    <frequency>43200</frequency>
    <directories check_all="yes" realtime="yes">/etc</directories>
    <directories check_all="yes" realtime="yes">/var/www</directories>
    <directories check_all="yes" realtime="yes">/home</directories>
  </syscheck>
  ```
- Testing the feature:
  ```bash
  # Modifying a monitored file
  echo "FIM test" | sudo tee -a /etc/hosts
  
  # Creating a file in /home
  touch /home/user/test_fim.txt
  ```
- **Expected result**: Alerts in the Dashboard → **File Integrity Monitoring** (added, modified, deleted)

---

## 📊 Wazuh Dashboard — Verification

| Module | Event Tested | Status |
|--------|--------------|--------|
| Application Logs | DVWA / Juice Shop access | ✅ |
| Attack Detection | Nmap scan, SQLi, Brute Force | ✅ |
| Vulnerability Detection | CVEs on Ubuntu packages | ✅ |
| File Integrity Monitoring | Modifications in `/etc`, `/home` | ✅ |

---

## 🛠️ Tools Used

- **Wazuh** (SIEM / HIDS)
- **Docker** & **Docker Compose**
- **DVWA** (Damn Vulnerable Web Application)
- **OWASP Juice Shop**
- **Kali Linux** (nmap, hydra, sqlmap, etc.)
- **Ubuntu Server**

---

## 🎯 Learning Objectives

- Understand how a SIEM/HIDS works (Wazuh)
- Deploy a secure lab infrastructure
- Simulate real attacks and analyze alerts
- Leverage the **Vulnerability Detection** and **FIM** modules

---

## 👤 Author

**Emna Khedher**
Project carried out as part of my training/intership during this summer of 2026

---

## 📚 References

- [Official Wazuh Documentation](https://documentation.wazuh.com/)
- [DVWA](https://github.com/digininja/DVWA)
- [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/)
- [Kali Linux Tools](https://www.kali.org/tools/)

---

## 📝 License

This project is licensed under the MIT License — see the `LICENSE` file for more details.

---
