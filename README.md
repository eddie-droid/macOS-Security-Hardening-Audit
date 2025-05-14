# macOS Security Hardening Audit (Personal System Lab)

This repository documents my hands-on system audit and hardening of a macOS laptop for cybersecurity learning purposes. I investigated root account status, scanned for open network ports, disabled vulnerable services, and automated defensive measures — all without turning off core protections like SIP.

---

## Goals

- Identify and disable unnecessary system services
- Lock down potential external access vectors
- Learn command-line auditing and service control using `launchctl`, `lsof`, `nano`, and `launchd`
- Avoid GUI tools and understand low-level macOS security behavior

---

## Steps Taken

### 1. Root User Audit
- ✅ Checked for the existence of the root user via `dscl`
- ✅ Verified ability to elevate via `sudo su` and then disabled root login using `dsenableroot -d`
- ✅ Verified root login was disabled by checking user shell in `/Users/root`

**Why it matters:** Prevents attackers from using a dormant but powerful account.

---

### 2. Open Port Scan & Service Review
- ✅ Used `lsof -i -n -P | grep LISTEN` and `netstat` to identify active ports
- ✅ Found services listening on 3306 (MySQL), 80 (Apache), 8021 (unknown)
- ✅ Determined which apps were bound to each port (e.g., Postman, Adobe, MySQL)

**Why it matters:** Every open port is a potential doorway into the system.

---

### 3. MySQL Hardening & Removal
- ✅ Created a custom `my.cnf` file to bind MySQL to `127.0.0.1`
- ✅ Verified changes using `lsof -i :3306`
- ✅ Removed MySQL completely when no longer needed
  - Uninstalled binary files
  - Removed user `_mysql`
  - Cleaned `/usr/local/mysql`, `/var/mysql`, and LaunchDaemons
- ✅ Confirmed `mysqld` process was killed and did not restart

**Why it matters:** MySQL is a high-value target — removing or restricting it significantly reduces attack surface.

---

### 4. Apache (httpd) Review and Control
- ✅ Identified Apache listening on port 80
- ✅ Stopped it using `sudo apachectl stop`
- ⚠️ Unable to unload its launch daemon due to macOS SIP (expected behavior)
- ✅ Created a `launchd` task to automatically stop Apache at login
- ✅ Wrote a script (`stop_apache.sh`) with elevated privileges
- ✅ Confirmed Apache no longer binds to port 80 on boot

**Why it matters:** Apache was running by default, opening a public-facing port unnecessarily.

---

### 5. Automation via LaunchAgent
- ✅ Wrote a `com.eddie.stopapache.plist` file in `~/Library/LaunchAgents`
- ✅ Configured to stop Apache on every boot without disabling SIP
- ✅ Used `launchctl load` to apply it immediately
- ✅ (Optional) Used `visudo` to allow the script to run `apachectl` without password prompt

---

## Key Commands Used

- `sudo lsof -i :PORT` – identify which process is listening
- `dscl . -read /Users/USERNAME` – read user account details
- `launchctl bootout` – unload system launch daemons (blocked by SIP)
- `nano`, `chmod +x`, `launchctl load` – build automation
- `sudo apachectl stop` – stop Apache manually
- `sudo kill PID` – stop lingering services

---

## What's Left / Next Steps

- [ ] Enable macOS firewall (next step)
- [ ] Log changes and scan ports after reboot
- [ ] Review outbound network traffic using tools like `Little Snitch` or `LuLu`

---

## Final Thoughts

This project taught me how to:
- Identify network exposures on macOS
- Understand the relationship between services, users, and permissions
- Respect macOS protections like SIP while still applying strong security practices
- Create defensive automation using native tools
