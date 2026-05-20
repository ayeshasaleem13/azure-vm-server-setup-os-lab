# azu# ☁️ Azure Cloud — Virtual Machine Setup & Server Configuration

> A two-phase Operating Systems lab project completed at **Fatima Jinnah Women University**.
> Two Ubuntu virtual machines are deployed on **Microsoft Azure Cloud**, networked together, and configured with five production-grade servers (FTP, Web, DNS, DHCP, Proxy).

---

**Student:** Amna Javed | **Reg No.:** 23-BCS-009
**Program:** BS Computer Science 5B | **Course:** Operating Systems
**Submitted To:** Sir Majid Shafique

---

## 📑 Table of Contents

- [Project Overview](#project-overview)
- [Phase 1 — VM Creation & Networking](#phase-1--vm-creation--networking)
  - [Tools & Software](#tools--software)
  - [Task 1 — Create VM1](#task-1--create-vm1)
  - [Task 2 — Create VM2](#task-2--create-vm2)
  - [Task 3 — Network Configuration](#task-3--network-configuration)
  - [Task 4 — Connectivity Test (Ping)](#task-4--connectivity-test-ping)
- [Phase 2 — Server Setup & Configuration](#phase-2--server-setup--configuration)
  - [1. FTP Server (vsftpd)](#1-ftp-server-vsftpd)
  - [2. Web Server (Apache2)](#2-web-server-apache2)
  - [3. DNS Server (BIND9)](#3-dns-server-bind9)
  - [4. DHCP Server (isc-dhcp-server)](#4-dhcp-server-isc-dhcp-server)
  - [5. Proxy Server (Squid)](#5-proxy-server-squid)
  - [6. Service Status Checks](#6-service-status-checks)
- [Architecture Summary](#architecture-summary)
- [Results](#results)
- [How to Reproduce](#how-to-reproduce)

---

## Project Overview

This lab demonstrates cloud-based OS virtualization using **Microsoft Azure** instead of traditional local hypervisors (VMware / VirtualBox). The project is split into two phases:

| Phase | Goal |
|---|---|
| Phase 1 | Create two Ubuntu VMs on Azure, configure networking, verify ping communication |
| Phase 2 | Install and configure 5 servers on VM1; test each from VM2 |

**Network:** Both VMs share the same Azure Virtual Network (`student-free-vm-vnet`)
**Server VM (VM1) IP:** `10.0.0.4`

---

## Phase 1 — VM Creation & Networking

### Tools & Software

| Tool | Purpose |
|---|---|
| Microsoft Azure Portal | VM creation and management |
| Azure for Students Subscription | Free-tier cloud access |
| Ubuntu Server 22.04 LTS | OS image for both VMs |
| Windows PowerShell | SSH access to VMs |
| Internet Browser | Portal access |

---

### Task 1 — Create VM1

Navigate to **Virtual Machines → Create → Azure Virtual Machine** and configure:

```
Subscription:         Azure for Students
Resource Group:       student-rg
VM Name:              student-free-vm
Region:               Central India
Image:                Ubuntu Server 22.04 LTS
Size:                 B2ats_v2 (Free tier eligible)
Authentication Type:  SSH public key
Username:             azureuser
SSH Key:              Generate new → Download .pem file
Inbound Port:         SSH (Port 22)
```

Click **Review + Create → Create**.

---

### Task 2 — Create VM2

Repeat the VM creation process with:

```
VM Name:              student-free-vm2
Resource Group:       student-rg  (same as VM1)
Virtual Network:      student-free-vm-vnet  (same as VM1)
SSH Key:              Generate new second key → Download
```

Both VMs are now deployed in the same resource group and virtual network.

---

### Task 3 — Network Configuration

Both VMs are connected to the **same Azure Virtual Network**:

```
Virtual Network:   student-free-vm-vnet
VM1 Private IP:    10.0.0.4
VM2 Private IP:    (assigned from same subnet)
```

No additional routing configuration is needed — Azure automatically enables internal communication between VMs in the same VNet.

---

### Task 4 — Connectivity Test (Ping)

SSH into VM1 from PowerShell:

```powershell
ssh -i <path-to-key1.pem> azureuser@<VM1-public-IP>
```

Ping VM2 from VM1:

```bash
ping <VM2-private-IP>
```

SSH into VM2 and ping VM1:

```powershell
ssh -i <path-to-key2.pem> azureuser@<VM2-public-IP>
```

```bash
ping 10.0.0.4
```

**Expected Result:** Successful ICMP replies in both directions — confirms network communication is working.

---

## Phase 2 — Server Setup & Configuration

All servers are installed and configured on **VM1** (`10.0.0.4`). Each server is verified from **VM2**.

---

### 1. FTP Server (vsftpd)

**Installation & Config:**

```bash
sudo apt install vsftpd -y
sudo nano /etc/vsftpd.conf
```

**Verification from VM2:**

```bash
ftp 10.0.0.4
# Login: ftpuser
# Password: ftp123
```

**Expected Output:**
```
230 Login successful.
```

**Status Check:**
```bash
sudo systemctl status vsftpd
# Active: active (running)
```

---

### 2. Web Server (Apache2)

**Installation:**

```bash
sudo apt install apache2 -y
```

**Verification from VM2:**

```bash
curl http://10.0.0.4
```

**Expected Output:** Apache2 default HTML page content returned.

**Status Check:**
```bash
sudo systemctl status apache2
# Active: active (running)
```

---

### 3. DNS Server (BIND9)

**Installation & Config:**

```bash
sudo apt install bind9 -y
sudo nano /etc/bind/named.conf.options
```

**Verification:**

```bash
dig @127.0.0.1 example.com
```

**Expected Output:** DNS resolution result with answer section showing IP for `example.com`.

**Status Check:**
```bash
sudo systemctl status bind9
# Active: active (running)
```

---

### 4. DHCP Server (isc-dhcp-server)

**Installation & Config:**

```bash
sudo apt install isc-dhcp-server -y
sudo nano /etc/dhcp/dhcpd.conf
```

**Config block:**

```
subnet 10.0.0.0 netmask 255.255.255.0 {
  range 10.0.0.10 10.0.0.100;
  option routers 10.0.0.4;
  option domain-name-servers 10.0.0.4;
}
```

**Verification from VM2:**

```bash
sudo dhclient -v
```

**Expected Output:** VM2 receives an IP address from the range `10.0.0.10 – 10.0.0.100`.

**Status Check:**
```bash
sudo systemctl status isc-dhcp-server
# Active: active (running)
```

---

### 5. Proxy Server (Squid)

**Installation & Config:**

```bash
sudo apt install squid -y
sudo nano /etc/squid/squid.conf
```

**Key config line:**
```
http_port 3128
```

**Verification from VM2:**

```bash
curl -x http://10.0.0.4:3128 http://example.com
```

**Expected Output:** `example.com` webpage HTML loads successfully through the proxy.

**Status Check:**
```bash
sudo systemctl status squid
# Active: active (running)
```

---

### 6. Service Status Checks

Run all status checks together to confirm every service is active:

```bash
sudo systemctl status vsftpd
sudo systemctl status apache2
sudo systemctl status bind9
sudo systemctl status isc-dhcp-server
sudo systemctl status squid
```

All should show: `Active: active (running)`

---

## Architecture Summary

```
┌─────────────────────────────────────────────┐
│          Azure Virtual Network               │
│           student-free-vm-vnet               │
│                                             │
│  ┌─────────────────┐   ┌─────────────────┐  │
│  │      VM1         │   │      VM2         │  │
│  │  10.0.0.4        │◄──►  (DHCP assigned) │  │
│  │  Ubuntu 22.04    │   │  Ubuntu 22.04    │  │
│  │                  │   │                  │  │
│  │  ✅ vsftpd (21)  │   │  Test client     │  │
│  │  ✅ Apache2 (80) │   │  for all servers │  │
│  │  ✅ BIND9 (53)   │   │                  │  │
│  │  ✅ DHCP (67)    │   │                  │  │
│  │  ✅ Squid (3128) │   │                  │  │
│  └─────────────────┘   └─────────────────┘  │
└─────────────────────────────────────────────┘
```

---

## Results

| Component | Status | Verified From |
|---|---|---|
| VM1 Created | ✅ Success | Azure Portal |
| VM2 Created | ✅ Success | Azure Portal |
| VM1 ↔ VM2 Ping | ✅ Success | PowerShell SSH |
| FTP Server (vsftpd) | ✅ Login successful (230) | VM2 |
| Web Server (Apache2) | ✅ Default page served | VM2 via curl |
| DNS Server (BIND9) | ✅ Name resolved | localhost dig |
| DHCP Server | ✅ IP assigned to VM2 | VM2 via dhclient |
| Proxy Server (Squid) | ✅ example.com loaded | VM2 via curl proxy |

---

## How to Reproduce

1. Create a free [Azure for Students](https://azure.microsoft.com/en-us/free/students/) account
2. Follow **Task 1** and **Task 2** to deploy both Ubuntu VMs in the same resource group and VNet
3. SSH into VM1 using PowerShell with your downloaded `.pem` key
4. Install each server in order (vsftpd → Apache2 → BIND9 → isc-dhcp-server → Squid)
5. After each install, SSH into VM2 and run the corresponding verification command
6. Run all `systemctl status` checks to confirm services are active

> **Note:** Azure Free Student tier has compute hour limits. Stop VMs when not in use to preserve your credits.

---

*Fatima Jinnah Women University | BS Computer Science | Operating Systems Lab*
*Submitted by: Amna Javed (23-BCS-009)*re-vm-server-setup-os-lab
