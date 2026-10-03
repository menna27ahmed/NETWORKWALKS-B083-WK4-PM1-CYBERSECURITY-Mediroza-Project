# Mediroza General Hospital — Black-Box Penetration Test

A full black-box penetration testing engagement against a simulated hospital web application, completed as part of the Networkwalks security training program. The project covers reconnaissance, authentication bypass via SQL injection, offline password cracking, metadata analysis, and the discovery of a critical data exposure on the target server.

> ⚠️ **Educational project.** This assessment was performed in a controlled training environment with explicit written authorization from Networkwalks. The target, data, and organization involved are simulated for training purposes. None of the techniques documented here should be used against any system without explicit written permission from its owner.

## 📋 Overview

| | |
|---|---|
| **Target** | Simulated hospital web application |
| **Engagement type** | Black-box penetration test |
| **Duration** | 5 days |
| **Findings** | 7 total — 3 Critical, 2 High, 2 Medium |

## 🎯 Attack Chain Summary

1. **Reconnaissance** — Ran `whois` and `dig` against the target domain, then checked `robots.txt`, which disclosed three hidden paths: `/patient/`, `/staff/`, and `/old/`.
2. **Username enumeration** — The patient login form returned different error messages for an invalid username versus a valid username with the wrong password, confirming `admin` as a real account.
3. **SQL injection → authentication bypass** — A single quote in the username field triggered a raw MySQL syntax error, confirming the field was injectable. The payload `admin' --` then bypassed authentication entirely.
4. **Data retrieval** — Logged in as `admin` and downloaded three encrypted patient lab report PDFs from the portal.
5. **Offline password cracking** — Extracted a hash from each PDF and cracked them offline. The first two fell instantly to a common top-100 password list. The third resisted that list and required a larger, extended wordlist to crack.
6. **Metadata analysis** — Decrypted the third report with `qpdf` and ran `exiftool` against it, revealing an internal comment left by IT staff referencing a database backup moved to a folder called `/old` before a site migration.
7. **Directory listing exposure** — Navigated to `/old/`, which was already flagged in `robots.txt`, and found directory listing enabled, exposing a full SQL database backup file.
8. **Critical data exposure** — Downloaded the backup and found plaintext staff salary records and confidential shareholder ownership data. The metadata author (`j.malik`) was cross-referenced against the staff table and confirmed to be the hospital's IT Systems Administrator, completing the attack chain from document metadata back to its source.

## 🛠️ Tools Used

- **whois / dig** — domain and DNS reconnaissance
- **curl** — fetching and reading `robots.txt`
- **Web browser (manual testing)** — login form behavior analysis
- **Manual SQL injection payloads** — authentication bypass
- **Online PDF hash extraction tool** — generating crackable hashes from encrypted PDFs
- **networkwalks cracking tool with wordlists** — recovering PDF passwords (default list, then an extended wordlist)
- **qpdf** — decrypting PDFs once passwords were recovered
- **exiftool** — extracting file metadata

## 📊 Findings Summary

| # | Vulnerability | Location | Risk |
|---|---|---|---|
| 1 | Username enumeration on login page | `/patient/login.php` | Medium |
| 2 | SQL injection — authentication bypass | `/patient/login.php` | **Critical** |
| 3 | Encrypted patient PDFs accessible post-bypass | `/patient/reports/` | High |
| 4 | Weak PDF passwords crackable via wordlist | `patient_report_*.pdf` | High |
| 5 | Sensitive metadata left in patient PDF | `patient_report_3.pdf` | Medium |
| 6 | Forgotten backup folder, directory listing enabled | `/old/` | **Critical** |
| 7 | Staff salaries & shareholder data exposed in plaintext | `/old/mediroza_db_backup_2019.sql` | **Critical** |


## ✅ Key Takeaways

- Individually minor issues — a verbose error message, a `robots.txt` entry, a forgotten backup — chained together into a critical data breach. Most real-world breaches happen this way, not through a single catastrophic flaw.
- Metadata left inside "finished" documents can quietly leak operational secrets. Always strip metadata before distributing files externally.
- A single unauthenticated, publicly listable directory turned an authentication bug into full exposure of HR and corporate ownership data — attack surface reviews need to include forgotten/legacy paths, not just the current application.

## 🔒 Disclaimer

This repository documents a simulated engagement completed as part of a cybersecurity training program (Networkwalks, Batch B083). All data, names, and the target domain referenced are fictional. Do not apply any of these techniques to a system you do not own or do not have explicit written authorization to test.

