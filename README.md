<div align="center">

# 🛡️ Q-Feeds Blocklist Update Scripts

**Automated malware IP blocklist updates for MikroTik RouterOS**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![RouterOS](https://img.shields.io/badge/RouterOS-7.15%2B-blue)](https://mikrotik.com/)

</div>

---

## 📋 Table of Contents

- [Quick Start](#-quick-start-installation)
- [Overview](#-overview)
- [How It Works](#-how-it-works)
- [Prerequisites](#-prerequisites)
- [Detailed Installation](#-detailed-installation-guide)
- [Configuration](#-configuration)
- [Usage](#-usage)
- [License](#-license)

---

## 🚀 Quick Start Installation

### Step 1: Acquire (free) API token from Q-Feeds
In order to use the blocklist you need to obtain an API-key which can be freely acquired on https://tip.qfeeds.com/ 

### Step 2: Copy the Scripts
1. Open `Malware Import FULL.txt` in this repository
2. Copy the entire script content
3. Open `Malware Import DIFF.txt` in this repository  
4. Copy the entire script content

### Step 3: Configure API Token
Before deploying, replace `XXXXXXXXXXXXXXX` with your Q-Feeds API token in both scripts:
- In `Malware Import FULL.txt`: Replace the token (Optionally add `&limit=xxxx` for memory limited devices)
- In `Malware Import DIFF.txt`: Replace the token

### Step 4: Deploy to RouterOS
1. Create a new RouterOS script named **"Malware Update - Full"** and paste the contents of `Malware Import FULL.txt`
2. Create a new RouterOS script named **"Malware Update - Diff"** and paste the contents of `Malware Import DIFF.txt`

### Step 5: Set Up Schedulers
Create two schedulers in RouterOS:

| Scheduler Name | Script | Recommended Schedule |
|---------------|--------|---------------------|
| `Malware Update - Full` | Malware Update - Full | Weekly or Daily |
| `Malware Update - Diff` | Malware Update - Diff | Every 20 minutes |

> **Note:** The full import script automatically manages the differential scheduler's state to prevent conflicts.

### Step 6: Run
The scripts will run automatically according to your scheduler settings. You can also run them manually from the RouterOS terminal or Winbox.

---

## 📖 Overview

This solution provides automated malware IP blocklist updates for MikroTik RouterOS devices using the Q-Feeds API. It uses a **two-script combination approach** that combines efficiency with reliability:

- **Full Import Script** - Complete blocklist refresh (runs less frequently)
- **Differential Update Script** - Incremental updates (runs frequently)

### Why Two Scripts?

- ✅ **Efficiency**: Differential updates are fast and use minimal resources
- ✅ **Reliability**: Full imports ensure complete synchronization
- ✅ **Automation**: Scripts coordinate automatically to prevent conflicts
- ✅ **Scalability**: Handles large blocklists efficiently

---

## 🔧 How It Works

### Full Import Script (`Malware Import FULL.txt`)

Performs a complete refresh of the malware IP blocklist:

```
┌─────────────────────────────────────────┐
│  1. Disable Differential Scheduler     │
│  2. Rename Existing Entries            │
│  3. Download All Pages (up to 199)      │
│  4. Import All IPs to "Malware-List"    │
│  5. Remove Old Entries                  │
│  6. Re-enable Differential Scheduler    │
└─────────────────────────────────────────┘
```

**When to run:** Weekly or daily for complete synchronization

### Differential Update Script (`Malware Import DIFF.txt`)

Performs incremental updates by processing only changes:

```
┌─────────────────────────────────────────┐
│  1. Download Differences Only           │
│  2. Add IPs prefixed with "+"           │
│  3. Remove IPs prefixed with "-"        │
│  4. Update "Malware-List"               │
└─────────────────────────────────────────┘
```

**When to run:** Every 20 minutes for quick updates

### Coordination Mechanism

The full import script automatically manages the differential scheduler:

- **Before starting**: Disables the differential update scheduler
- **After completion**: Re-enables the differential update scheduler

This ensures:
- 🔒 No conflicts between scripts
- ⚡ Efficient resource usage
- 🔄 Always up-to-date blocklist

---

## ✅ Prerequisites

Before installing, ensure you have:

- [x] **RouterOS 7.15+** (tested on 7.15 and 7.17+ and 7.20.6)
- [x] **Q-Feeds API Token** - Get yours free at [tip.qfeeds.com](https://tip.qfeeds.com/)
- [x] **Admin Privileges** - Scripts require full admin rights
- [x] **Firewall Address List** - Create "Malware-List" (or scripts will create it)

---

## 📝 Detailed Installation Guide

### 1. Get Your API Token

Visit [https://tip.qfeeds.com/](https://tip.qfeeds.com/) to obtain your free Q-Feeds API token.

### 2. Copy the Scripts

Simply view the script files in this repository and copy their contents:
- **`Malware Import FULL.txt`** - Full blocklist import script
- **`Malware Import DIFF.txt`** - Differential update script

You can view them directly on GitHub or download the repository files if you prefer.

### 3. Configure the Scripts

Before deploying, replace the API token placeholder in both scripts:

```routeros
:local apitoken "YOUR_API_TOKEN_HERE";
```

**What to change:**
- In `Malware Import FULL.txt` (line 3): Replace `XXXXXXXXXXXXXXX` with your token
  - Optional: Add `&limit=xxxx` parameter for memory-limited devices
- In `Malware Import DIFF.txt` (line 3): Replace `XXXXXXXXXXXXXXX` with your token

### 4. Deploy to RouterOS

#### Option A: Using Winbox
1. Open Winbox and connect to your RouterOS device
2. Go to **System** → **Scripts**
3. Click **+** to create a new script
4. Name it **"Malware Update - Full"**
5. Paste the contents of `Malware Import FULL.txt`
6. Repeat for **"Malware Update - Diff"** using `Malware Import DIFF.txt`

#### Option B: Using WebFig
1. Open WebFig in your browser
2. Navigate to **System** → **Scripts**
3. Create new scripts as described above

#### Option C: Using Terminal/SSH
```routeros
/system script add name="Malware Update - Full" source={...paste content...}
/system script add name="Malware Update - Diff" source={...paste content...}
```

### 4. Set Up Schedulers

#### Create Full Import Scheduler
```routeros
/system scheduler add name="Malware Update - Full" \
    start-time=startup interval=1d \
    on-event="Malware Update - Full"
```

#### Create Differential Update Scheduler
```routeros
/system scheduler add name="Malware Update - Diff" \
    start-time=startup interval=20m \
    on-event="Malware Update - Diff"
```

**Recommended schedules:**
- **Full Import**: Weekly (`interval=7d`) or Daily (`interval=1d`)
- **Differential**: Every 20 minutes (`interval=20m`)

---

## ⚙️ Configuration

### Firewall Address List

Both scripts work with the **"Malware-List"** firewall address list. The scripts will automatically create this list if it doesn't exist.

To manually create it:
```routeros
/ip firewall address-list add list="Malware-List" address=0.0.0.0 comment="Placeholder"
/ip firewall address-list remove [find list="Malware-List" address="0.0.0.0"]
```

### Customizing the List Name

If you want to use a different list name, update the following in both scripts:
- Replace `list="Malware-List"` with your desired list name

---

## 🎯 Usage

### Manual Execution

Run scripts manually from RouterOS terminal:
```routeros
/system script run "Malware Update - Full"
/system script run "Malware Update - Diff"
```

### Monitoring

Check logs to monitor script execution:
```routeros
/log print where topics~"info" and message~"Malware"
```

### Troubleshooting

**Scripts not running?**
- Verify API token is correct
- Check scheduler is enabled: `/system scheduler print`
- Ensure admin privileges are granted

**Import errors?**
- Check internet connectivity
- Verify Q-Feeds API is accessible
- Review RouterOS logs for specific errors

---

## 📄 License

This project is licensed under the **MIT License** - see the [LICENSE](LICENSE) file for details.

---

## ⚠️ Disclaimer

**Use at your own risk.**

Please test these scripts in your environment before deploying them in production. The author is not responsible for any issues or damages that may occur from their use.

---

<div align="center">



[Report Bug](https://github.com/Q-Feeds/Q-Feeds-Integration-for-Mikrotik-RouterOS/issues) · [Request Feature](https://github.com/Q-Feeds/Q-Feeds-Integration-for-Mikrotik-RouterOS/issues)

</div>
