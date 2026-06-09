# Azure VM Remote Connection – Lab Report

> **Assignment:** Configure, connect to, and secure Azure Virtual Machines using RDP (Windows) and SSH (Linux).  
> **Submitted by:** [Your Name]  
> **Date:** [Submission Date]  
> **Program:** 3MTT / [Your Institution]

---

## Table of Contents

1. [Overview](#overview)
2. [Task 1 – Configure Networking and Security (NSG)](#task-1--configure-networking-and-security-nsg)
3. [Task 2 – Identify Connection Endpoints](#task-2--identify-connection-endpoints)
4. [Task 3 – Establish a Windows RDP Connection](#task-3--establish-a-windows-rdp-connection)
5. [Task 4 – Establish a Linux SSH Connection](#task-4--establish-a-linux-ssh-connection)
6. [Task 5 – Verify System Access](#task-5--verify-system-access)
7. [Task 6 – Secure the Environment](#task-6--secure-the-environment)
8. [Task 7 – Manage Remote Sessions](#task-7--manage-remote-sessions)
9. [NSG Configuration Summary](#nsg-configuration-summary)
10. [Troubleshooting](#troubleshooting)
11. [Screenshots](#screenshots)

---

## Overview

This lab demonstrates the end-to-end process of remotely connecting to Azure Virtual Machines (VMs) — both Windows and Linux — by configuring Network Security Groups (NSGs), retrieving public IP addresses, establishing authenticated sessions, verifying system access, hardening inbound rules, and safely terminating sessions.

---

## Task 1 – Configure Networking and Security (NSG)

### Steps Taken

1. Navigated to **Azure Portal → Virtual Machines → [VM Name] → Networking**.
2. Opened the linked **Network Security Group (NSG)**.
3. Selected **Inbound security rules → Add**.
4. Created the following rules:

| Rule Name         | Priority | Protocol | Port  | Source          | Action |
|-------------------|----------|----------|-------|-----------------|--------|
| Allow-RDP         | 300      | TCP      | 3389  | My IP (initial) | Allow  |
| Allow-SSH         | 310      | TCP      | 22    | My IP (initial) | Allow  |

5. Saved the rules and confirmed they appeared under **Inbound security rules**.

> **Note:** Initial rules were set to `Any` source for testing, then restricted to specific IPs in Task 6.

---

## Task 2 – Identify Connection Endpoints

### Steps Taken

1. Navigated to **Azure Portal → Virtual Machines → [VM Name] → Overview**.
2. Located the **Public IP address** field in the Essentials panel.
3. Recorded the public IP addresses:

| VM Name          | OS      | Public IP Address  |
|------------------|---------|--------------------|
| [Windows VM Name]| Windows | `[e.g. 20.x.x.x]` |
| [Linux VM Name]  | Linux   | `[e.g. 52.x.x.x]` |

> **Screenshot:** See [screenshots/public-ip.png](screenshots/public-ip.png)

---

## Task 3 – Establish a Windows RDP Connection

### Steps Taken

1. Opened **Remote Desktop Connection** (`mstsc`) on the local Windows machine  
   *(or used Microsoft Remote Desktop on macOS/Linux).*
2. Entered the Windows VM's Public IP address in the **Computer** field.
3. Clicked **Connect**, then entered the administrator credentials:
   - **Username:** `[azureuser or custom admin name]`
   - **Password:** `[defined at VM creation]`
4. Accepted the certificate warning and confirmed the connection.
5. The Windows desktop loaded successfully inside the RDP session.

> **Screenshot:** See [screenshots/rdp-connection.png](screenshots/rdp-connection.png)

---

## Task 4 – Establish a Linux SSH Connection

### Method A – Terminal (SSH Key-Based)

```bash
ssh -i ~/.ssh/[your-key].pem azureuser@[Linux VM Public IP]
```

### Method B – Password Authentication via Terminal

```bash
ssh azureuser@[Linux VM Public IP]
# Enter password when prompted
```

### Method C – PuTTY (Windows)

1. Opened **PuTTY**.
2. Entered the Linux VM's Public IP under **Host Name**.
3. Set **Port** to `22` and **Connection type** to `SSH`.
4. (For key auth) Navigated to **Connection → SSH → Auth** and loaded the `.ppk` private key.
5. Clicked **Open** and logged in with `azureuser`.

> **Screenshot:** See [screenshots/ssh-connection.png](screenshots/ssh-connection.png)

---

## Task 5 – Verify System Access

### Windows VM

After connecting via RDP:
- Opened **File Explorer** and browsed `C:\Users\` and `C:\Program Files\`.
- Launched **Command Prompt** and ran:

```cmd
systeminfo
ipconfig /all
```

- Opened **Server Manager** to inspect installed roles and features.

> **Screenshot:** See [screenshots/windows-system-access.png](screenshots/windows-system-access.png)

### Linux VM

After connecting via SSH:
- Navigated the file system:

```bash
ls /home/
ls /etc/
```

- Checked installed packages and system info:

```bash
uname -a
lsb_release -a
dpkg -l        # Debian/Ubuntu
# OR
rpm -qa        # RHEL/CentOS
```

- Checked running processes and disk usage:

```bash
top
df -h
```

> **Screenshot:** See [screenshots/linux-system-access.png](screenshots/linux-system-access.png)

---

## Task 6 – Secure the Environment

### Steps Taken

1. Determined local public IP using [https://whatismyip.com](https://whatismyip.com) or:

```bash
curl ifconfig.me
```

2. Returned to **Azure Portal → NSG → Inbound security rules**.
3. Edited the **Allow-RDP** and **Allow-SSH** rules:
   - Changed **Source** from `Any` to `IP Addresses`.
   - Entered the specific source IP (e.g., `197.x.x.x/32`).
4. Saved the updated rules.
5. Verified that connection from the whitelisted IP still succeeded.
6. Confirmed that access from a different IP was blocked.

> This step ensures only authorized hosts can initiate remote sessions, significantly reducing the attack surface.

---

## Task 7 – Manage Remote Sessions

### Windows RDP – Terminating the Session

- Clicked **Start → [User Icon] → Sign out** to properly end the session (preferred over just closing the window, which leaves the session active).
- Alternatively, in the RDP window toolbar, clicked **X** and chose **Disconnect**.

### Linux SSH – Terminating the Session

```bash
exit
# OR
logout
```

> Proper session termination ensures no idle sessions consume VM resources or present a security risk.

---

## NSG Configuration Summary

| Rule Name   | Priority | Protocol | Port | Source IP / CIDR   | Direction | Action |
|-------------|----------|----------|------|--------------------|-----------|--------|
| Allow-RDP   | 300      | TCP      | 3389 | `[Your IP]/32`     | Inbound   | Allow  |
| Allow-SSH   | 310      | TCP      | 22   | `[Your IP]/32`     | Inbound   | Allow  |
| DenyAllInbound | 65500 | Any     | Any  | Any                | Inbound   | Deny   |

> The default `DenyAllInbound` rule (Azure default) blocks all traffic not explicitly allowed by higher-priority rules.

---

## Troubleshooting

| Issue | Cause | Resolution |
|-------|-------|------------|
| RDP connection timeout | Port 3389 not open in NSG | Added Allow-RDP inbound rule |
| SSH: `Connection refused` | VM not yet started or SSH daemon inactive | Restarted VM; verified `sshd` service |
| SSH: `Permission denied (publickey)` | Wrong key file or username | Confirmed key path and used `azureuser` |
| Can't find Public IP | VM may use dynamic IP | Assigned a static public IP in Azure portal |
| RDP certificate warning | Self-signed cert on VM | Accepted warning; expected behavior in lab environments |

---

## Screenshots

Place your screenshot files in a `screenshots/` folder in this repository.

```
screenshots/
├── public-ip.png               # Azure portal showing VM public IP
├── nsg-rules.png               # NSG inbound rules configuration
├── rdp-connection.png          # Active Windows RDP desktop session
├── ssh-connection.png          # Active Linux SSH terminal session
├── windows-system-access.png   # systeminfo / ipconfig output on Windows VM
├── linux-system-access.png     # uname -a / df -h output on Linux VM
└── nsg-restricted.png          # NSG rules after IP restriction (Task 6)
```

> Replace placeholder values (IPs, VM names, dates) with your actual lab data before submission.
