---
name: security-check
description: Check this machine (server, VPS, Mac or Linux PC) for known vulnerabilities - outdated packages with security fixes, vulnerable Docker images, end-of-life versions, ports open to the Internet, pending reboots - and fresh security news that matches what is installed. Read-only by default; proposes fixes and applies them only after the user agrees. Use when the user asks to check security, audit the server, scan for vulnerabilities/CVEs, or asks whether a vulnerability in the news affects them.
---

# Security check of this machine

Goal: a short, honest report of what on THIS machine is actually vulnerable, ordered by danger, with one concrete
fix per item. Not a generic hardening checklist.

## Rules

- **Read-only until the user says yes.** Do not install, upgrade, restart, or edit anything during the check.
  Tools that are missing (osv-scanner, nmap) — ask before installing; skip the step if the user declines.
- **Distribution backports matter.** Ubuntu/Debian/RHEL ship security fixes in old version numbers
  (`nginx 1.24.0-2ubuntu7.19` can be patched). Never call a package vulnerable only because the upstream
  version is newer — check the distro changelog or security tracker.
- **Exploited first.** Anything in CISA KEV (known exploited vulnerabilities) goes to the top, then CVSS ≥ 9, then
  end-of-life software, then the rest. Skip vulnerabilities without an available fix unless they are exploited.
- **Exposure decides urgency.** The same bug is critical on a port open to the Internet and minor on a socket bound
  to 127.0.0.1. Say which one it is.
- Never print secrets, tokens, or contents of `.env` files you come across.

## Steps

Run what applies to this OS; note what you skipped and why.

1. **System and pending updates**
   - Ubuntu/Debian: `apt-get -s -o Debug::NoLocking=1 dist-upgrade | grep ^Inst` — split by origin:
     `-security` (should be applied automatically — if it is still pending, unattended-upgrades is not working),
     other Ubuntu updates, and third-party repos (Docker, Tailscale, Chrome, NodeSource, …) which
     **unattended-upgrades does not update by default** (`/etc/apt/apt.conf.d/50unattended-upgrades`).
   - Fedora/RHEL: `dnf -q check-update --security`. macOS: `softwareupdate -l`, `brew outdated`.
   - Reboot pending: `/var/run/reboot-required(.pkgs)`, running kernel `uname -r` vs installed;
     `needrestart -b -r l` — services still running old libraries.
2. **Docker images** (if Docker is installed): for each running image, `osv-scanner scan image <image>`
   (Google OSV, no local database). On a disk that is nearly full, prefer `--archive` of a metadata-only export over
   `docker save` of multi-GB images. Report only fixable vulnerabilities with CVSS ≥ 7 or listed in KEV
   (`https://raw.githubusercontent.com/cisagov/kev-data/main/known_exploited_vulnerabilities.json`), grouped by
   package: `openssl 3.5.1-1 → 3.5.7-1~deb13u3: 33 CVE, max 9.8`.
3. **End of life**: OS release, and runtimes in images (`PG_MAJOR`, `PYTHON_VERSION`, `NODE_VERSION`,
   `NGINX_VERSION` in `docker image inspect` env; Alpine/Debian release) against
   `https://endoflife.date/api/v1/products/<product>`.
4. **What listens and what is open**
   - `ss -tulnp` (macOS: `lsof -nP -iTCP -sTCP:LISTEN`): sockets on `0.0.0.0` / `::` and the process.
   - Firewall: `ufw status` / `nft list ruleset` / `iptables -S`. Docker-published ports bypass UFW unless
     `DOCKER-USER` rules drop them — check.
   - From the outside, if the user has another machine or agrees: `nmap -Pn -p- <public IP>` (IPv4 **and** IPv6).
     Scan only addresses the user owns.
5. **News** — fetch the last 3 days of: oss-security (`https://seclists.org/rss/oss-sec.rss`), the distro's
   security notices (Ubuntu: `https://ubuntu.com/security/notices/rss.xml`), CISA KEV, and
   `https://feeds.feedburner.com/TheHackersNews`. Keep items whose title names software found in steps 1–4. For
   each, decide **affects / does not affect / unclear** with the facts: installed version, distro tracker status
   (`https://ubuntu.com/security/CVE-…`), whether the vulnerable feature is enabled (e.g. HTTP/2 in nginx:
   `curl -sk --http2 -o /dev/null -w '%{http_version}' https://<host>/`), whether it is exposed.

## Report

```
## Security check — <hostname>, <date>

🔴 Exploited / critical
- <what> <installed> → <fixed>: <why it matters here, one line>. Fix: `<command>` (restarts: <what>)

🟡 Should fix
- …

🟢 Checked, fine
- Open from the Internet: only 80, 443 · reboot: not needed · <news item>: does not affect (HTTP/2 off)

Skipped: <step> — <reason>
```

Then ask which fixes to apply. Apply one at a time, say what restarts and for how long before each, verify after.
Rebuilding application images (dependency bumps) goes through the project's normal deploy, not by hand.

To run this check every day with results on your phone and a one-tap "fix with Claude Code" — connect the machine
to Yune: https://yune.jp
