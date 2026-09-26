# Legacy Server Defense & Hardening (Incident Response & Remediation)

## 📌 Project Overview
This project simulates a real-world incident response and server hardening engagement. The objective was to detect unauthorized network scans against a vulnerable legacy test server (`192.168.4.3`), preserve volatile forensic evidence, contain active root backdoors without system downtime, and implement permanent remediation and continuous monitoring.

* **Environment:** VirtualBox Isolated Cyber Range (NAT Network)
* **Target Node:** Legacy Linux Server (`192.168.4.3`)
* **Analyst Node:** Kali Linux
* **Framework Alignment:** NIST Incident Response Lifecycle, MITRE ATT&CK

---

## 🔒 Target Vulnerability Matrix

| Vector / Port | Vulnerability Identified | CVE ID | Action Taken & Remediation |
| :--- | :--- | :--- | :--- |
| **Port 21 (FTP)** | vsftpd 2.3.4 Backdoor | CVE-2011-2523 | Package purged, legacy binaries removed, firewall rules updated. |
| **Port 139/445 (SMB)** | Samba `usermap_script` RCE | CVE-2007-2447 | Updated Samba release, disabled legacy SMBv1 protocols. |
| **Port 1524 (Bindshell)** | Root Bindshell Backdoor | N/A | Process terminated (`kill -9`), listener and binaries removed. |
| **Port 3632 (distccd)** | Unauthenticated RCE | CVE-2004-2687 | Package purged, public network access restricted via firewall. |
| **Port 6667 (UnrealIRCd)**| UnrealIRCd Backdoor RCE | CVE-2010-2075 | Daemon removed, system dependencies updated. |

---

## 🚀 Incident Response Workflow

### Phase 1: Preparation & Reconnaissance
* Deployed a closed-loop lab environment via VirtualBox NAT network.
* Executed targeted `nmap` service version scanning to identify active attack surfaces.

### Phase 2: Detection & AI-Assisted Threat Profiling
* Correlated open service versions with NVD and MITRE databases to confirm zero-day and legacy backdoors.
* Utilized AI-assisted workflows to accelerate initial Threat Profile generation, manually validating all findings against official CVE entries.

### Phase 3: Containment & Forensic Evidence Preservation
* **Volatile Memory Preservation:** Captured running processes (`ps aux`), socket connections (`netstat`, `ss -tulnp`), and active sessions (`w`) to `/tmp/evidence/` prior to mitigation.
* **Surgical Containment:** Terminated malicious processes via PID-specific execution (`kill -9`), avoiding full system shutdown and maintaining zero operational downtime.
* **Verification:** Probed closed vectors using `nc -v` to ensure access denial.

### Phase 4: Eradication, Recovery & Hardening
* Permanently uninstalled vulnerable packages (`vsftpd`, `samba`, `distccd`, `unrealircd`).
* Applied OS patches, removed persistence scripts, and disabled unnecessary system daemons adhering to the principle of least privilege.
* Deployed an automated Bash health check script to continuously monitor target ports.

---

## 🛠️ Automated Health Check Script (`health_check.sh`)

```bash
#!/bin/bash
# Continuous Monitoring Script for Hardened Host
TARGET="192.168.4.3"
PORTS=(21 139 445 1524 3632 6667)
SECURE=1

echo "[*] Running security health check against $TARGET..."
for p in "${PORTS[@]}"; do
  nc -z -w2 "$TARGET" "$p" 2>/dev/null
  if [ $? -eq 0 ]; then
    echo "[ALERT] Port $p/tcp is STILL OPEN/VULNERABLE!"
    SECURE=0
  else
    echo "[OK] Port $p/tcp is closed/filtered."
  fi
done

if [ $SECURE -eq 1 ]; then
  echo "STATUS: SECURE"
  exit 0
else
  echo "STATUS: BREACHED/MISCONFIGURED"
  exit 2
fi
```
