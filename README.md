# 🛡️ PrestaShop Antimalware Protection Script

> **Automatic protection system against Credit Card Stealer malware for PrestaShop**  
> Detects and removes malware that steals credit card data during checkout, restores infected files, and scans for fake images containing malicious code.

---

## 🌍 Language / Idioma

Choose your preferred language to read the complete documentation:

<div align="center">

[![Español](https://img.shields.io/badge/Español-red?style=for-the-badge&logo=data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMTIwMCIgaGVpZ2h0PSI4MDAiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyI+DQo8cmVjdCB3aWR0aD0iMTIwMCIgaGVpZ2h0PSI4MDAiIGZpbGw9IiNjNjBiMWUiLz4NCjxyZWN0IHdpZHRoPSIxMjAwIiBoZWlnaHQ9IjUzMy4zMyIgZmlsbD0iI2ZmYzQwMCIgeT0iMTMzLjMzIi8+DQo8L3N2Zz4=)](README_ES.md)
&nbsp;&nbsp;&nbsp;&nbsp;
[![English](https://img.shields.io/badge/English-blue?style=for-the-badge&logo=data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMTIwMCIgaGVpZ2h0PSI2MDAiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyI+DQo8cmVjdCB3aWR0aD0iMTIwMCIgaGVpZ2h0PSI2MDAiIGZpbGw9IiMwMTJxNjkiLz4NCjxwYXRoIGQ9Ik0wLDBMMTIwMCw2MDBNMTIwMCwwTDAsNjAwIiBzdHJva2U9IiNmZmYiIHN0cm9rZS13aWR0aD0iMTIwIi8+DQo8cGF0aCBkPSJNMCwwTDEyMDAsNjAwTTEyMDAsUEwwLDYwMCIgc3Ryb2tlPSIjYzgxMDJlIiBzdHJva2Utd2lkdGg9IjgwIi8+DQo8L3N2Zz4=)](README_EN.md)

</div>

---

## 📝 Quick Overview

### What does this script do?

✅ **Detects malware** in critical PHP files (Controller.php, FrontController.php, AdminLoginController.php)  
✅ **Automatically restores** infected files from secure URLs  
✅ **Scans fake images** in `/img/` directory (malware disguised as PNG/JPG files)  
✅ **Removes malicious payloads** automatically  
✅ **Creates backups** before any modification  
✅ **Email notifications** when detecting problems  
✅ **Runs automatically** via cron every minute (configurable)  
✅ **Detailed logging** of all operations  

### How does it work?

1. **Scans 3 critical PHP files** looking for specific malware patterns
2. **Downloads clean files** from URLs you configure (from official PrestaShop)
3. **Replaces infected files** with clean versions (after creating backups)
4. **Scans /img/ directory** using 4 detection methods for fake images
5. **Sends email alerts** when infections are detected
6. **Logs everything** for monitoring and forensic analysis

### Protection against:

- 🔴 **Credit Card Stealer** - Malware that steals card data during checkout
- 🔴 **Backdoors** in PHP core files
- 🔴 **Fake images** containing JavaScript/PHP code
- 🔴 **Data exfiltration** to Chinese servers (106.14.40.200, 47.102.208.65)

---

## 🚀 Quick Start

```bash
# 1. Configure the script (edit lines 14-39)
nano prestashop_antimalware.sh

# 2. Upload to server
scp prestashop_antimalware.sh user@server:/home/user/security/

# 3. Give execution permissions
chmod +x /home/user/security/prestashop_antimalware.sh

# 4. Test manually
bash /home/user/security/prestashop_antimalware.sh

# 5. Configure cron (every minute)
crontab -e
* * * * * /bin/bash /home/user/security/prestashop_antimalware.sh >> /home/user/logs/cron_output.log 2>&1
```

---

## 📚 Complete Documentation

For detailed installation instructions, configuration, troubleshooting, and FAQ:

### 📖 [Read Full Documentation in Spanish](README_ES.md)

### 📖 [Read Full Documentation in English](README_EN.md)

---

## 📦 What's Included

- **`prestashop_antimalware.sh`** - Main protection script (500+ lines)
- **`README_ES.md`** - Complete documentation in Spanish
- **`README_EN.md`** - Complete documentation in English
- **Clean PHP files** - Controller.php, FrontController.php, AdminLoginController.php

---

## ⚡ Key Features

| Feature | Description |
|---------|-------------|
| **🔍 Malware Detection** | Scans for 8 specific patterns in PHP files |
| **🔄 Auto Restoration** | Downloads and replaces infected files automatically |
| **🖼️ Fake Image Detection** | 4 detection methods (file type, patterns, PNG/JPEG headers, Base64) |
| **💾 Automatic Backups** | Creates timestamped backups before any change |
| **📧 Email Alerts** | Notifications when infections are detected |
| **📊 Detailed Logging** | Log rotation at 10MB, multiple log levels |
| **⚙️ Highly Configurable** | 9 variables to customize behavior |
| **🔒 Security First** | Validates files before replacement, restrictive permissions |

---

## ⚠️ Important Notes

- ⚠️ **PHP files must be renamed to .txt** on your server to serve as plain text (see documentation)
- ⚠️ This protects against **this specific malware** - not a replacement for updating PrestaShop
- ⚠️ **Change ALL passwords** after cleaning the infection
- ⚠️ **GDPR compliance:** Notify affected customers if their data was compromised

---

## 📞 Support

For issues, questions, or contributions:

1. Check the troubleshooting section in the full documentation
2. Review the logs: `/home/user/logs/prestashop_antimalware.log`
3. Execute manually to see detailed error messages

---

## 📝 Version

**Version:** 1.0  
**Date:** February 6, 2026  
**License:** Provided "as is" without warranties

---

<div align="center">

**Choose your language to continue:**

**[📖 Español](README_ES.md)** | **[📖 English](README_EN.md)**

</div>
