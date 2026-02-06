# 🛡️ PrestaShop Antimalware Protection Script

**Version:** 1.0  
**Date:** February 6, 2026  
**Author:** Developed for automatic protection against Credit Card Stealer

---

## 🌍 Language / Idioma

<div align="center">

[![Back](https://img.shields.io/badge/⬅️_Back_to_Home-gray?style=for-the-badge)](README_ANTIMALWARE.md)
&nbsp;&nbsp;
[![Español](https://img.shields.io/badge/🇪🇸_Español-red?style=for-the-badge)](README_ES.md)

</div>

---

## 📋 TABLE OF CONTENTS

1. [Description](#description)
2. [Features](#features)
3. [Requirements](#requirements)
4. [Installation](#installation)
5. [Configuration](#configuration)
6. [Usage](#usage)
7. [Cron Configuration](#cron-configuration)
8. [Logs and Monitoring](#logs-and-monitoring)
9. [Troubleshooting](#troubleshooting)
10. [Security](#security)
11. [FAQ](#faq)

---

## 🎯 DESCRIPTION

This script automatically protects your PrestaShop installation against **"Credit Card Stealer"** malware that:

- Infects core PHP files (Controller.php, FrontController.php)
- Steals credit card data during checkout
- Disguises itself as fake images in the `/img/` directory
- Sends stolen data to servers in China

**The script:**
- ✅ Automatically detects infections
- ✅ Restores infected files from secure URLs
- ✅ Removes malware disguised as images
- ✅ Creates backups before modifying files
- ✅ Sends email notifications when detecting problems
- ✅ Runs automatically via cron

---

## ⚙️ FEATURES

### PHP Malware Detection
- Searches for specific patterns in critical files:
  - `jschecks`, `order_llx`, Base64-encoded IPs
  - References to C&C servers (106.14.40.200, 47.102.208.65)
  - Obfuscated code characteristic of the stealer

### Automatic Restoration
- Downloads clean files from configurable URLs
- Verifies that downloaded files are valid PHP
- Creates automatic backups before any modification
- Preserves correct permissions (644)

### Fake Image Detection
Detects disguised malware using **4 methods**:
1. **File type analysis:** Detects ASCII files marked as PNG/JPG
2. **Pattern search:** Identifies JavaScript/PHP code in images
3. **PNG header validation:** Verifies magic headers (89504e47)
4. **JPEG header validation:** Verifies magic headers (ffd8)

### Logging System
- Detailed log of all operations
- Automatic rotation at 10MB
- Log levels: INFO, ALERT, SUCCESS, ERROR, WARNING

### Notifications
- Automatic email when infection is detected
- Optional: email when everything is clean
- Compatible with `mail` and `sendmail`

---

## 📦 REQUIREMENTS

### Required software:
- ✅ Bash shell (Linux/Unix)
- ✅ curl (to download files)
- ✅ grep, sed, awk (standard utilities)
- ✅ SSH access to server
- ✅ Write permissions on PrestaShop
- ⚠️ OPTIONAL: mail/sendmail (for notifications)

### Verify requirements:
```bash
# Verify curl is available
which curl

# Verify bash
which bash

# Verify write permissions
touch /home/user/test.txt && rm /home/user/test.txt
```

---

## 🚀 INSTALLATION

### Step 1: Prepare clean PrestaShop files

**IMPORTANT:** PHP files cannot be served directly for download because the server executes them as code. You must rename them to `.txt`.

1. **Download your exact PrestaShop version:**
   ```bash
   # From GitHub (example for 1.7.8.7)
   wget https://github.com/PrestaShop/PrestaShop/archive/refs/tags/1.7.8.7.zip
   unzip 1.7.8.7.zip
   cd PrestaShop-1.7.8.7
   ```

2. **Extract the 3 required files:**
   ```bash
   mkdir ~/prestashop_clean
   cp classes/controller/Controller.php ~/prestashop_clean/
   cp classes/controller/FrontController.php ~/prestashop_clean/
   cp controllers/admin/AdminLoginController.php ~/prestashop_clean/
   ```

3. **Rename to .txt to serve as plain text:**
   ```bash
   cd ~/prestashop_clean
   mv Controller.php Controller.php.txt
   mv FrontController.php FrontController.php.txt
   mv AdminLoginController.php AdminLoginController.php.txt
   ```

4. **Upload to an accessible URL:**
   
   **Option A - Subdirectory on your hosting:**
   ```bash
   # Create protected directory
   mkdir -p /home/user/public_html/clean_files
   
   # Upload files
   cp ~/prestashop_clean/*.txt /home/user/public_html/clean_files/
   
   # Final URLs:
   # https://yourdomain.com/clean_files/Controller.php.txt
   # https://yourdomain.com/clean_files/FrontController.php.txt
   # https://yourdomain.com/clean_files/AdminLoginController.php.txt
   ```

   **Option B - Secure external hosting (RECOMMENDED):**
   - Upload to a separate server
   - Use a subdomain: `https://clean.yourdomain.com/`
   - Protect with .htaccess if necessary

### Step 2: Upload script to server

```bash
# Via SCP
scp prestashop_antimalware.sh user@yourserver.com:/home/user/security/

# Via FTP/SFTP
# Use your preferred FTP client (FileZilla, WinSCP, etc.)
```

### Step 3: Give execution permissions

```bash
chmod +x /home/user/security/prestashop_antimalware.sh
```

### Step 4: Create required directories

```bash
# Log directory
mkdir -p /home/user/logs

# Backup directory
mkdir -p /home/user/var/backups/infected

# Temporary directory
mkdir -p /home/user/public_html/var/files

# Verify permissions
chmod 755 /home/user/logs
chmod 755 /home/user/var/backups/infected
chmod 755 /home/user/public_html/var/files
```

---

## ⚙️ CONFIGURATION

Edit the script and modify the variables in **lines 14-39**:

```bash
nano /home/user/security/prestashop_antimalware.sh
```

### Variables to configure:

```bash
# === CONFIGURATION - EDIT THESE VALUES ===

# 1. Path to your PrestaShop installation
PRESTASHOP_ROOT="/home/user/public_html"

# 2. URLs of clean files (IMPORTANT: must end in .txt)
CLEAN_CONTROLLER_URL="https://yourdomain.com/clean_files/Controller.php.txt"
CLEAN_FRONTCONTROLLER_URL="https://yourdomain.com/clean_files/FrontController.php.txt"
CLEAN_ADMINLOGIN_URL="https://yourdomain.com/clean_files/AdminLoginController.php.txt"

# 3. Email for notifications
EMAIL_ALERTS="admin@yourdomain.com"

# 4. Send email when infection is detected?
SEND_EMAIL_ON_INFECTION="yes"

# 5. Send email when everything is clean?
SEND_EMAIL_ON_CLEAN="no"  # Recommended "no" if using cron every minute

# 6. Log location
LOG_FILE="/home/user/logs/prestashop_antimalware.log"
LOG_MAX_SIZE=10485760  # 10MB

# 7. Backup directory
BACKUP_DIR="/home/user/var/backups/infected"

# 8. Temporary directory for downloads
TEMP_DIR="/home/user/public_html/var/files"

# 9. Verify SSL certificates
VERIFY_SSL="yes"  # Change to "no" if you have SSL problems
```

### Complete configuration example:

```bash
PRESTASHOP_ROOT="/home/sc1gijo7672/public_html"
CLEAN_CONTROLLER_URL="https://rlm.llc/ps/1787/Controller.php.txt"
CLEAN_FRONTCONTROLLER_URL="https://rlm.llc/ps/1787/FrontController.php.txt"
CLEAN_ADMINLOGIN_URL="https://rlm.llc/ps/1787/AdminLoginController.php.txt"
EMAIL_ALERTS="admin@mystore.com"
SEND_EMAIL_ON_INFECTION="yes"
SEND_EMAIL_ON_CLEAN="no"
LOG_FILE="/home/sc1gijo7672/logs/prestashop_antimalware.log"
BACKUP_DIR="/home/sc1gijo7672/var/backups/infected"
TEMP_DIR="/home/sc1gijo7672/public_html/var/files"
VERIFY_SSL="yes"
```

---

## 🎮 USAGE

### Manual Execution

```bash
# Run the script manually
bash /home/user/security/prestashop_antimalware.sh
```

**Expected output if everything is clean:**
```
════════════════════════════════════════════════════════════
  PrestaShop Antimalware Protection - Scan Started
  2026-02-06 18:05:53
════════════════════════════════════════════════════════════

[1/4] Checking Controller.php...
[2/4] Checking FrontController.php...
[3/4] Checking AdminLoginController.php...
[4/4] Scanning /img/ directory...
ℹ Scanning /img/ directory (non-recursive)...
✓ /img/ directory clean

════════════════════════════════════════════════════════════
  SCAN SUMMARY
════════════════════════════════════════════════════════════
✓ SYSTEM CLEAN
  No malware detected
════════════════════════════════════════════════════════════
```

**Output if infection is detected:**
```
════════════════════════════════════════════════════════════
  PrestaShop Antimalware Protection - Scan Started
  2026-02-06 18:05:53
════════════════════════════════════════════════════════════

[1/4] Checking Controller.php...
⚠ INFECTION DETECTED: Controller.php
↻ Downloading clean file: Controller.php
✓ Backup saved: /home/.../Controller.php.infected.20260206_180554
✓ Successfully restored: Controller.php

[2/4] Checking FrontController.php...
⚠ INFECTION DETECTED: FrontController.php
↻ Downloading clean file: FrontController.php
✓ Backup saved: /home/.../FrontController.php.infected.20260206_180554
✓ Successfully restored: FrontController.php

[3/4] Checking AdminLoginController.php...
[4/4] Scanning /img/ directory...
ℹ Scanning /img/ directory (non-recursive)...
⚠ MALWARE DETECTED: fake_logo.jpg
✓ Removed: fake_logo.jpg

════════════════════════════════════════════════════════════
  SCAN SUMMARY
════════════════════════════════════════════════════════════
⚠ INFECTION DETECTED AND CLEANED
  Infected PHP files: 2
  Restored PHP files: 2
  Malware files removed: 1

  Infected files have been backed up at:
  /home/user/var/backups/infected
════════════════════════════════════════════════════════════
```

---

## ⏰ CRON CONFIGURATION

For automatic 24/7 protection, configure a cron job.

### Option A: Via cPanel

1. Access **cPanel → Cron Jobs**
2. Configure frequency:
   - **Every minute (RECOMMENDED):**
     - Minute: `*`
     - Hour: `*`
     - Day: `*`
     - Month: `*`
     - Weekday: `*`
   
3. **Command:**
   ```bash
   /bin/bash /home/user/security/prestashop_antimalware.sh >> /home/user/logs/cron_output.log 2>&1
   ```

4. **Save**

### Option B: Via SSH (crontab)

```bash
# Edit crontab
crontab -e

# Add one of these lines:

# Every minute (RECOMMENDED for maximum protection)
* * * * * /bin/bash /home/user/security/prestashop_antimalware.sh >> /home/user/logs/cron_output.log 2>&1

# Every 5 minutes
*/5 * * * * /bin/bash /home/user/security/prestashop_antimalware.sh >> /home/user/logs/cron_output.log 2>&1

# Every 15 minutes
*/15 * * * * /bin/bash /home/user/security/prestashop_antimalware.sh >> /home/user/logs/cron_output.log 2>&1

# Business hours only (9am-6pm, weekdays)
* 9-18 * * 1-5 /bin/bash /home/user/security/prestashop_antimalware.sh >> /home/user/logs/cron_output.log 2>&1
```

### Verify cron is active:

```bash
# List active crons
crontab -l

# Wait 1-2 minutes and check log
tail -f /home/user/logs/cron_output.log
```

---

## 📊 LOGS AND MONITORING

### View logs in real time:

```bash
# Main script log
tail -f /home/user/logs/prestashop_antimalware.log

# Cron output
tail -f /home/user/logs/cron_output.log
```

### Search for detected infections:

```bash
# View all alerts
grep "ALERT" /home/user/logs/prestashop_antimalware.log

# View statistics
grep "STATS" /home/user/logs/prestashop_antimalware.log

# View last 50 lines
tail -50 /home/user/logs/prestashop_antimalware.log
```

### View created backups:

```bash
# List backups
ls -lah /home/user/var/backups/infected/

# View content of a backup
less /home/user/var/backups/infected/Controller.php.infected.20260206_180554
```

### Clean old logs:

```bash
# View log size
du -h /home/user/logs/prestashop_antimalware.log

# Rotate manually if too large
mv /home/user/logs/prestashop_antimalware.log \
   /home/user/logs/prestashop_antimalware.log.old
touch /home/user/logs/prestashop_antimalware.log
```

---

## 🔧 TROUBLESHOOTING

### Problem 1: "Permission denied"

**Symptom:**
```
bash: /home/user/security/prestashop_antimalware.sh: Permission denied
```

**Solution:**
```bash
chmod +x /home/user/security/prestashop_antimalware.sh
```

---

### Problem 2: "Error downloading clean file"

**Symptom:**
```
✗ Error downloading clean file
```

**Cause:** PHP files are being executed instead of downloaded.

**Solution:**
1. **Make sure URLs end in .txt:**
   ```bash
   CLEAN_CONTROLLER_URL="https://yourdomain.com/files/Controller.php.txt"
   # NOT: Controller.php
   ```

2. **Verify files are renamed on the server:**
   ```bash
   # On the server where you hosted the files
   ls -la /path/to/clean_files/
   # You should see: Controller.php.txt, FrontController.php.txt, etc.
   ```

3. **Manual test:**
   ```bash
   curl https://yourdomain.com/files/Controller.php.txt | head -20
   # Should show PHP code, NOT execute it
   ```

---

### Problem 3: "No such file or directory" in temp directory

**Symptom:**
```
head: cannot open '/home/user/public_html/var/files/ps_clean_123.php'
```

**Cause:** Temporary directory doesn't exist or lacks permissions.

**Solution:**
```bash
# Create directory
mkdir -p /home/user/public_html/var/files
chmod 755 /home/user/public_html/var/files

# Verify TEMP_DIR is configured correctly in the script
grep TEMP_DIR /home/user/security/prestashop_antimalware.sh
```

---

### Problem 4: "curl: command not found"

**Symptom:**
```
bash: curl: command not found
```

**Solution:**
```bash
# Ubuntu/Debian
sudo apt-get install curl

# CentOS/RedHat
sudo yum install curl

# If you don't have root access, contact your hosting provider
```

---

### Problem 5: Not receiving emails

**Symptom:** Script works but notifications don't arrive.

**Solution:**

1. **Verify mail is installed:**
   ```bash
   which mail
   # If it returns nothing:
   sudo apt-get install mailutils
   ```

2. **Change configuration:**
   ```bash
   # If you don't need emails, disable them:
   SEND_EMAIL_ON_INFECTION="no"
   ```

3. **Manual test:**
   ```bash
   echo "Test email" | mail -s "Test" your@email.com
   ```

---

### Problem 6: "SSL certificate problem"

**Symptom:**
```
curl: (60) SSL certificate problem: unable to get local issuer certificate
```

**Solution:**
```bash
# In the script, change:
VERIFY_SSL="no"
```

---

### Problem 7: Temporary files accumulating

**Symptom:** The `var/files/` directory has many `ps_clean_*.php` files.

**Cause:** If curl fails, temporary files aren't cleaned up.

**Solution:**
```bash
# Clean temporary files manually
rm -f /home/user/public_html/var/files/ps_clean_*.php

# Or add to cron (automatically cleans daily):
0 0 * * * find /home/user/public_html/var/files/ps_clean_*.php -mtime +1 -delete 2>/dev/null
```

---

## 🔒 SECURITY

### Protecting clean files

If you host clean files on your own server, protect them:

```bash
# Create .htaccess in the directory
cat > /home/user/public_html/clean_files/.htaccess << 'EOF'
# Allow access only from your server
Order Deny,Allow
Deny from all
Allow from 123.456.789.012
# Replace with your PrestaShop server IP
EOF
```

### Recommended permissions

```bash
# Protection script
chmod 500 /home/user/security/prestashop_antimalware.sh

# Backup directory
chmod 700 /home/user/var/backups/infected

# Logs
chmod 644 /home/user/logs/prestashop_antimalware.log
```

### Security backups

```bash
# Backups are automatically saved with timestamp:
# Controller.php.infected.20260206_180554
# FrontController.php.infected.20260206_180554
# fake_logo.jpg.malware.20260206_180555

# IMPORTANT: Backups contain malware, don't execute them
# They're only for forensic analysis or recovery if there's a false positive
```

---

## ❓ FAQ

### Why do I need to rename PHP files to .txt?

Web servers execute `.php` files as code. If you try to download `Controller.php`, the server will execute it and return its output (usually empty or an error), not the source code. By renaming it to `.txt`, the server serves it as plain text.

### Is it safe to keep backups of infected files?

Yes, they're just text files and don't execute automatically. They're useful for:
- Forensic analysis
- Legal evidence
- Recovery if there's a false positive

You can manually delete them whenever you want:
```bash
rm -rf /home/user/var/backups/infected/*
```

### How often should the script run?

**Recommendation:** Every 1 minute for maximum protection.

The script is very lightweight (< 1 second execution) and only sends email when it detects problems.

### What happens if the malware reinfects?

The script will detect and clean the infection automatically in the next cycle (maximum 1 minute if using cron every minute).

**However**, you should investigate:
- How did the attacker get in?
- Is FTP/SSH access compromised?
- Are there vulnerable modules installed?
- Are passwords weak?

### Does it protect against all infections?

NO. This script specifically protects against the analyzed "Credit Card Stealer" malware. For complete protection:
- Update PrestaShop to the latest version
- Use strong passwords
- Keep modules updated
- Implement WAF (Cloudflare)
- Make regular backups

### Can I modify the script?

Yes, the script is fully modifiable. Main function documentation:

- `detect_malware_in_php()` - Detects malicious patterns
- `restore_file_from_url()` - Downloads and restores files
- `is_fake_image()` - Detects fake images
- `scan_img_directory()` - Scans /img/

### Does it work with any PrestaShop version?

The script works with any version, BUT you need to use clean files from your exact PrestaShop version.

### What to do after the first cleanup?

1. ✅ Change ALL passwords (FTP, SSH, DB, Admin)
2. ✅ Review users with server access
3. ✅ Audit installed modules
4. ✅ Update PrestaShop if possible
5. ✅ Implement WAF (free Cloudflare)
6. ✅ Configure automatic backups
7. ✅ Monitor logs for 2 weeks

---

## 📞 SUPPORT

### Check system status

```bash
# Run manually
bash /home/user/security/prestashop_antimalware.sh

# View last 100 log lines
tail -100 /home/user/logs/prestashop_antimalware.log

# Search for errors
grep "ERROR" /home/user/logs/prestashop_antimalware.log

# Verify cron is working
crontab -l
```

### Important files

- **Script:** `/home/user/security/prestashop_antimalware.sh`
- **Main log:** `/home/user/logs/prestashop_antimalware.log`
- **Cron log:** `/home/user/logs/cron_output.log`
- **Backups:** `/home/user/var/backups/infected/`
- **Temporary files:** `/home/user/public_html/var/files/`

---

## 📝 CHANGELOG

### Version 1.0 (2026-02-06)
- ✅ Malware detection in 3 critical PHP files
- ✅ Automatic restoration from configurable URLs
- ✅ Fake image detection with 4 methods
- ✅ Automatic backup system
- ✅ Detailed logs with rotation
- ✅ Email notifications
- ✅ Configurable variables (including TEMP_DIR)
- ✅ Automatic creation of required directories
- ✅ Improved PHP file validation (first 20 lines)
- ✅ Support for .txt files as clean source

---

## 📜 LICENSE

This script is provided "as is" without warranties of any kind.
Use at your own risk.

---

## ✅ INSTALLATION CHECKLIST

- [ ] Clean PrestaShop files downloaded
- [ ] Files renamed to .txt
- [ ] Files uploaded to accessible URL
- [ ] Script uploaded to server
- [ ] Execution permissions configured (chmod +x)
- [ ] Directories created (logs, backups, temp)
- [ ] Script configured (all variables edited)
- [ ] Manual test successfully executed
- [ ] Cron job configured
- [ ] Test email received (if applicable)
- [ ] Logs verified for 24 hours
- [ ] Passwords changed
- [ ] PrestaShop update plan prepared

---

<div align="center">

**Your PrestaShop is now protected against automatic reinfections!** 🛡️

[![Back to Home](https://img.shields.io/badge/⬅️_Back_to_Home-gray?style=for-the-badge)](README_ANTIMALWARE.md)
&nbsp;&nbsp;
[![Versión en Español](https://img.shields.io/badge/🇪🇸_Leer_en_Español-red?style=for-the-badge)](README_ES.md)

</div>
