# DeployMania

**A lightweight, self-hosted control panel for your Ubuntu server, by DCodeMania.**

Install it on your VPS with one command, then manage everything from the browser: software,
sites and domains with free SSL, push-to-deploy from GitHub, databases, cron jobs, workers
and users. It's a single small program (about 16 MB) that runs as a system service.

> **Early release.** DeployMania is new. Try it on a fresh or test server before you rely
> on it for production sites.

## What you can do

- **Software:** install and manage Nginx, PHP 8.1–8.5 (with extensions), Node.js,
  MySQL/MariaDB, PostgreSQL, Redis, Supervisor, PM2, Composer, uv and Git. Software
  that's already installed is detected.
- **Sites:**
  - Static, PHP, Laravel, Node.js, Python and reverse-proxy sites.
  - Each site runs under its own Linux user.
  - Domains, aliases and redirects, with free Let's Encrypt SSL that renews itself.
- **Deploys:**
  - Connect your personal and organization GitHub accounts, or use a per-site deploy key.
  - Deploy automatically on push, or by hand.
  - Zero-downtime releases, with build and post-deploy scripts.
  - Shared `storage/` and `.env`, and one-click rollback.
  - Edit `.env` in the browser.
- **Databases:** MySQL/MariaDB and PostgreSQL databases and users, with generated
  passwords and access levels.
- **Background jobs:** cron jobs with plain-English schedules and logs, and Supervisor
  workers, with one-click Laravel scheduler and queue worker.
- **People and access:** invite people with roles (owner, admin, developer, viewer), use
  two-factor login, and manage Linux users, SSH keys and sudo.
- **Updates and backups:** signed one-click updates, and daily backups of the panel's own
  data.
- **Existing servers welcome:** sites, databases, cron jobs and workers that were already
  on the server show up in the panel, and DeployMania never changes things it didn't
  create.

## Requirements

- Ubuntu 22.04 or 24.04, or Debian 12 (64-bit Intel/AMD or ARM).
- Root access (or `sudo`).
- Port **8443** open for the panel, and ports **80** and **443** for your sites.

## Install

```bash
curl -fsSL https://github.com/DCodeMania/deploymania-releases/releases/latest/download/install.sh | sudo bash
```

The installer checks the download's signature and checksum, installs DeployMania as a
system service and prints:

- a one-time **setup link** to create your owner account;
- the fingerprint of the panel's certificate.

Open the link (`https://YOUR_SERVER_IP:8443/…`). Your browser will warn about the
self-signed certificate: compare the fingerprint, then continue.

Options:

```bash
# use another port for the panel
curl -fsSL https://github.com/DCodeMania/deploymania-releases/releases/latest/download/install.sh | sudo bash -s -- --port 9443

# install a specific version
curl -fsSL https://github.com/DCodeMania/deploymania-releases/releases/latest/download/install.sh | sudo bash -s -- --version v0.1.0
```

If your provider has its own firewall (cloud firewall or security group), allow TCP 8443
there too. The installer only opens the port in `ufw`.

**Next step:** give the panel its own domain in **Settings → Panel domain**. It gets a
trusted certificate, the browser warning goes away, and connecting GitHub for
push-to-deploy needs it.

## Updating

DeployMania checks for new releases twice a day. Owners update with one click in
**Settings → Updates**. Or, on the server:

```bash
sudo deploymania update            # install the latest release
sudo deploymania update --check    # only check
```

Before every update, the panel backs up its database and verifies the release's signature
and checksum. It then restarts in a few seconds; your sites keep running.

If an update ever goes wrong, go back to the previous version:

```bash
sudo mv /usr/local/bin/deploymania.previous /usr/local/bin/deploymania && sudo systemctl restart deploymania
```

## Backups of the panel

The panel backs up its own database every day (kept for 7 days) and before every update,
in `/var/lib/deploymania/backups`. Owners can download backups from **Settings → Backups**.
They contain the panel's secrets, so keep downloaded copies somewhere safe.

```bash
sudo deploymania backup                # make a backup now
sudo deploymania backup list           # list backups
sudo deploymania restore <name>        # restore one (the panel restarts)
```

These back up DeployMania's own data. Back up your sites' files and databases as well.

## Useful commands

| Command | What it does |
|---|---|
| `sudo deploymania admin setup-url` | Show the setup link again |
| `sudo deploymania admin reset-password --email you@example.com` | Set a new password for a panel user (add `--disable-2fa` if needed) |
| `sudo deploymania setup` | Repair the service (safe to run again) |
| `systemctl status deploymania` | Service status |
| `journalctl -u deploymania -f` | Follow the logs |
| `deploymania version` | Show the installed version |

## Verify a download yourself

Every release has `checksums.txt`, signed with DeployMania's release key
(`checksums.txt.sig`). The installer checks both automatically; to check by hand:

```bash
base=https://github.com/DCodeMania/deploymania-releases/releases/latest/download
curl -fsSLO $base/checksums.txt -fsSLO $base/checksums.txt.sig -fsSLO $base/deploymania_linux_amd64

cat > release.pem <<'EOF'
-----BEGIN PUBLIC KEY-----
MCowBQYDK2VwAyEAs1gmTl/8bPqxTLuGikpSyTxJyXrnMFEL5EEuYz56yOE=
-----END PUBLIC KEY-----
EOF

openssl pkeyutl -verify -pubin -inkey release.pem -rawin -in checksums.txt -sigfile checksums.txt.sig
sha256sum --ignore-missing -c checksums.txt
```

## Uninstall

```bash
sudo systemctl disable --now deploymania
sudo rm /etc/systemd/system/deploymania.service /usr/local/bin/deploymania
sudo rm -rf /etc/deploymania /var/lib/deploymania   # also delete the panel's data
```

Your sites, databases and installed software are not removed.

## Help

Found a problem or have a question? Open an issue in this repository.

---

© DCodeMania
