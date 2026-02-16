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
1. Open `Malware Import FULL.rsc` in this repository
2. Copy the entire script content
3. Open `Malware Import DIFF.rsc` in this repository  
4. Copy the entire script content

### Step 3: Configure API Token and Settings
Before deploying, configure both scripts:

**API Token:**
- Replace `XXXXXXXXXXXXXXX` with your Q-Feeds API token in both scripts
- The FULL script automatically uses pagination (4,000 IPs per page) and stops when all data is fetched

**Dynamic Storage Option (Recommended):**
- Set `:local useDynamic "yes";` to store entries in RAM only (prevents flash wear)
- Set `:local useDynamic "no";` to write entries to disk (persistent across reboots)
- Default is `"yes"` - recommended for frequent updates to prevent flash wear

### Step 4: Deploy to RouterOS
1. Create a new RouterOS script named **"Malware Update - Full"** and paste the contents of `Malware Import FULL.rsc`
2. Create a new RouterOS script named **"Malware Update - Diff"** and paste the contents of `Malware Import DIFF.rsc`

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

### Full Import Script (`Malware Import FULL.rsc`)

Performs a complete refresh of the malware IP blocklist:

```
┌──────────────────────────────────────────┐
│  1. Disable Differential Scheduler       │
│  2. Rename Existing Entries              │
│  3. Download Pages Automatically         │
│     (4,000 IPs per page, stops when      │
│      no more data available)             │
│  4. Import All IPs to "Malware-List"     │
│  5. Validate Import Success              │
│  6. Remove Old Entries (if successful)   │
│  7. Rollback on Failure (if needed)      │
│  8. Re-enable Differential Scheduler     │
└──────────────────────────────────────────┘
```

**Features:**
- ✅ **Automatic Pagination**: Fetches data in pages of 4,000 IPs and stops automatically when complete
- ✅ **Automatic Rollback**: Restores previous list if import fails
- ✅ **IPv4 & IPv6 Support**: Handles both IP address formats (RouterOS validates addresses)
- ✅ **CIDR Support**: Supports CIDR notation (e.g., `192.168.1.0/24`)
- ✅ **Efficient API Usage**: Only makes necessary API calls, stops early when done

**When to run:** Weekly or daily for complete synchronization

### Differential Update Script (`Malware Import DIFF.rsc`)

Performs incremental updates by processing only changes:

```
┌─────────────────────────────────────────┐
│  1. Download Differences Only           │
│  2. Add IPs prefixed with "+"           │
│  3. Remove IPs prefixed with "-"        │
│  4. Update "Malware-List"               │
└─────────────────────────────────────────┘
```

**Features:**
- ✅ **IPv4 & IPv6 Support**: Handles both IP address formats
- ✅ **CIDR Support**: Supports CIDR notation
- ✅ **Input Validation**: Proper IP address validation

**When to run:** Every 20 minutes for quick updates

### Coordination Mechanism

The full import script automatically manages the differential scheduler:

- **Before starting**: Disables the differential update scheduler
- **After completion**: Re-enables the differential update scheduler

This ensures:
- 🔒 No conflicts between scripts
- ⚡ Efficient resource usage
- 🔄 Always up-to-date blocklist

### Security Features

The scripts include several security and reliability features:

- ✅ **Address Validation**: RouterOS validates all IP addresses when adding them (invalid addresses are automatically skipped)
- ✅ **CIDR Support**: Handles both plain IPs and CIDR notation (e.g., `192.168.1.0/24`)
- ✅ **Automatic Rollback**: Restores previous list if import fails (prevents loss of protection)
- ✅ **Flash Wear Protection**: Optional RAM-only storage prevents flash degradation
- ✅ **Error Handling**: Comprehensive error handling with logging
- ✅ **Efficient Pagination**: Automatically fetches data in optimal page sizes and stops when complete

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
- **`Malware Import FULL.rsc`** - Full blocklist import script
- **`Malware Import DIFF.rsc`** - Differential update script

You can view them directly on GitHub or download the repository files if you prefer.

### 3. Configure the Scripts

Before deploying, configure both scripts:

**API Token:**
```routeros
:local apitoken "YOUR_API_TOKEN_HERE";
```

**What to change:**
- In `Malware Import FULL.rsc` (line 3): Replace `XXXXXXXXXXXXXXX` with your token
  - The script automatically uses pagination (4,000 IPs per page) to handle large datasets efficiently
- In `Malware Import DIFF.rsc` (line 3): Replace `XXXXXXXXXXXXXXX` with your token

**Dynamic Storage Option (Line 6):**
```routeros
:local useDynamic "yes";  # Recommended: RAM only (prevents flash wear)
```

- **`"yes"`** (Recommended): Stores entries in RAM only. Entries are lost on reboot but repopulated automatically. Prevents flash wear from frequent writes.
- **`"no"`**: Writes entries to disk. Entries persist across reboots but can cause flash wear with frequent updates (every 20 minutes).

### 4. Deploy to RouterOS

#### Option A: Using Winbox
1. Open Winbox and connect to your RouterOS device
2. Go to **System** → **Scripts**
3. Click **+** to create a new script
4. Name it **"Malware Update - Full"**
5. Paste the contents of `Malware Import FULL.rsc`
6. Repeat for **"Malware Update - Diff"** using `Malware Import DIFF.rsc`

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

### Dynamic Storage Option

The scripts include a configurable `useDynamic` variable that controls how address-list entries are stored:

**RAM Storage (Recommended - `useDynamic="yes"`):**
- ✅ **Prevents Flash Wear**: Entries stored in RAM only, never written to disk
- ✅ **Ideal for Frequent Updates**: Perfect for scripts running every 20 minutes
- ✅ **Large Lists**: No config file bloat, even with thousands of entries
- ⚠️ **Reboot Behavior**: Entries are lost on reboot but automatically repopulated by schedulers
- **Use Case**: Recommended for most deployments, especially with frequent updates

**Disk Storage (`useDynamic="no"`):**
- ✅ **Persistent**: Entries survive reboots without waiting for script execution
- ⚠️ **Flash Wear Risk**: Frequent writes (every 20 minutes) can wear out RouterBoard flash
- ⚠️ **Config Bloat**: Large lists increase config file size (limited flash space)
- **Use Case**: Only recommended for infrequent updates or when persistence is critical

**Configuration:**
```routeros
# In both scripts, line 6:
:local useDynamic "yes";  # Set to "yes" (RAM) or "no" (disk)
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
- Verify API URL is accessible (check internet connectivity)

**Import errors?**
- Check internet connectivity
- Verify Q-Feeds API is accessible: `https://api.qfeeds.com`
- Review RouterOS logs for specific errors: `/log print where topics~"info" and message~"Malware"`
- Check if rollback occurred (indicates import failure)

**Rollback occurred?**
- The script automatically restores the previous list if import fails
- Check logs for rollback messages: `/log print where message~"rollback"`
- Verify API connectivity and token validity
- Ensure sufficient memory/resources available

**Empty list after reboot?**
- If using `useDynamic="yes"` (default), entries are RAM-only and lost on reboot
- This is normal - entries will be repopulated when schedulers run
- To persist across reboots, set `useDynamic="no"` (not recommended for frequent updates)

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
