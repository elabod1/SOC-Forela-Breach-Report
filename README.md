# Forela Forum Breach - Incident Report

## 📌 Executive Summary
On April 25th and 26th, 2023, an external contractor compromised the internal Forela forum. The attacker accessed the forum via the Guest Wi-Fi, created a rogue account, exploited a third-party LDAP extension to bypass authentication, and successfully exfiltrated the forum's database. The attack was facilitated through a mix of social engineering (a phishing post) and misconfigured security settings (plaintext LDAP credentials).

## 🎯 Indicators of Compromise (IOCs)
| Indicator Type | Value | Description |
| :--- | :--- | :--- |
| **IP Address** | `10.10.0.78` | Attacker's primary IP (Guest Wi-Fi) |
| **IP Address** | `10.255.254.2` | Secondary IP used for Admin access |
| **Username** | `apoole1` (User ID: 52) | Attacker's rogue account |
| **Extension** | `rokx/dborldap` | Malicious/Exploited LDAP extension |
| **Malicious URL** | `http://10.10.0.78/update.php` | Credential harvesting endpoint |
| **File** | `backup_1682506471_dcsr71p7fyijoyq8.sql.gz` | Stolen database backup |

## ⏱️ Timeline of Events (UTC)
*Note: Log times were recorded in UTC+1 and have been converted to UTC.*

*   **25/04/2023 11:07:39** - Attacker (`10.10.0.78`) starts reconnaissance on the forum.
*   **25/04/2023 11:08:19** - Attacker registers a new account (`apoole1`).
*   **25/04/2023 11:09:02** - Successful login to the Administration Control Panel (ACP) from `10.255.254.2`.
*   **25/04/2023 11:46:07** - Attacker enables the `rokx/dborldap` extension.
*   **25/04/2023 11:47:31** - Attacker modifies Authentication settings (`mode=auth`) to use LDAP.
*   **25/04/2023 11:48:06** - Attacker modifies a user account (User ID: 51), likely adding them to the Admin group.
*   **25/04/2023 12:17:22** - Attacker posts a phishing thread titled "Hello Everyone" (Post ID: 9) containing a fake login form.
*   **26/04/2023 10:52:11** - Attacker logs back into ACP to exfiltrate data.
*   **26/04/2023 11:01:38** - Attacker downloads the full database backup from `/store/`.

## 🔍 Technical Analysis (Cyber Kill Chain)

### 1. Reconnaissance & Weaponization
The contractor connected to the Guest Wi-Fi (`10.10.0.78`) and prepared a credential harvester (`update.php`), along with a fake HTML login form.

### 2. Delivery (Phishing)
The attacker registered the account `apoole1` and published Post ID: 9 ("Hello Everyone"). The post contained hidden HTML creating a fake login prompt. (Confirmed via CyberChef extraction).

### 3. Exploitation & Installation
The attacker used a stolen admin session (`10.255.254.2`) to access the ACP. They installed and enabled the `rokx/dborldap` extension and changed the forum's authentication method to LDAP. 
**Critical Finding:** The `phpbb_config` database table stored the LDAP password (`password1`) in **plaintext**. This allowed the attacker to bypass standard authentication.

### 4. Actions on Objectives (Exfiltration)
On April 26th, the attacker used the compromised admin access to navigate to the `/store/` directory and downloaded the database backup file (`backup_...sql.gz`).

## 🛡️ Recommendations
1.  **Containment:** Block IPs `10.10.0.78` and `10.255.254.2`. Disable the `rokx/dborldap` extension immediately.
2.  **Credential Reset:** Force password resets for all Admin accounts and the LDAP server.
3.  **Eradication:** Delete Post ID 9 and delete the rogue user `apoole1`.
4.  **Hardening:** 
    *   Never store LDAP passwords in plaintext; use environment variables or a secrets manager.
    *   Restrict access to the `/store/` directory via web server configuration (e.g., Nginx/Apache).
    *   Enforce 2FA for all ACP logins.
