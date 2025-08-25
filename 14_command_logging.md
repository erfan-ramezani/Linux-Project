# 14 – Command Logging & Real-Time Alerts

## Requirements
- Log **every command** users execute, with timestamp, user, TTY, and CWD.
- Show logs in a dedicated file: `/var/log/command_audit.log`.
- If a **non-admin** user runs a command, send an **immediate email** alert to the admin.
- Only admin can read logs; others cannot read or delete them.
- The log file must be **append-only** (no truncation or deletion), even by root unless attributes are changed.
- Debian 12 / Ubuntu 24.04.

## Overview
We use:
- A global shell hook via `/etc/profile.d/` that logs each interactive command using `trap DEBUG`.
- `logger` with a custom tag (`cmd_audit`) → `rsyslog` routes entries to `/var/log/command_audit.log`.
- A helper script sends **real-time email alerts** for commands typed by **non-admin** users.
- Permissions + `chattr +a` enforce append-only and restrict access.

> Note: This captures interactive shells (bash/sh). Non-interactive executions (e.g., some cron/systemd services) are out of scope unless similarly instrumented. For kernel-level coverage use `auditd` (optional, not required here).

---

## Steps

### 1) Install dependencies (logger, mail)
```bash
sudo apt update
sudo apt install -y rsyslog mailutils msmtp-mta coreutils
sudo systemctl enable --now rsyslog
