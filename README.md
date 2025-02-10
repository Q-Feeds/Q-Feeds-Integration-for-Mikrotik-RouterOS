#Q-Feeds Blocklist Update Script

This repository contains a MikroTik RouterOS script that automates the update of your Q-Feeds blocklist. The script downloads the latest Q-Feeds blocklist, cleans up previous entries, and imports the new addresses into your firewall, keeping your security measures up-to-date without manual intervention.

Overview

The script performs the following steps:

File Cleanup: Removes any previously downloaded Q-Feeds.rsc file.
Download Blocklist: Fetches the latest Q-Feeds blocklist from the Q-Feeds API.
File Verification: Waits until the blocklist file is fully downloaded.
Firewall Cleanup: Removes old entries from your Q-Feeds-BlockList firewall address list.
Blocklist Import: Imports the new blocklist into your RouterOS configuration.
Logging: Logs each step to help you monitor progress and troubleshoot if necessary.
Prerequisites

RouterOS Version: Tested on RouterOS 7.15 and 7.17+.
API Token: You need a valid Q-Feeds API token. Be sure to update the script with your token.
Admin Privileges: The script must be executed with full admin rights on your RouterOS device.
Installation and Usage

Clone the Repository:
Clone this repository to your local machine or transfer it to your MikroTik device using your preferred method (FTP, Winbox, etc.).
Review the Script:
Open the script.txt file in this repository to view the full script and customize it as needed (e.g., update the API token).
Deploy the Script:
Copy the contents of script.txt into a new script on your MikroTik device using Winbox, the WebFig interface, or via the command line.
Execution:
Run the script manually or schedule it with the RouterOS scheduler to automate blocklist updates.
License

This project is licensed under the MIT License.

Disclaimer

Use at your own risk.
Please test this script in your environment before deploying it in production. The author is not responsible for any issues or damages that may occur from its use.