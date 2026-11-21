# 05 — Fail2ban: watching the door

**Fail2ban** reads the logs and, when an IP fails to log in several times, blocks it in the firewall for a while. It's one way to stop **brute-force attacks** (trying passwords over and over until one works).

I use two jails: one for **SSH** and one for the **Proxmox web UI** (port 8006), following the [official Proxmox guide](https://pve.proxmox.com/wiki/Fail2ban).

## Install

```bash
sudo apt update
sudo apt install fail2ban
```

## Proxmox VE 9: no `auth.log`

Proxmox VE 9 doesn't install a syslog daemon, so there's no `/var/log/auth.log`. Everything goes to the **systemd journal**, and fail2ban has to read from there: that's what `backend = systemd` does.

## Configure

Copy the base config to your own file. Package updates can overwrite `jail.conf`, but never `jail.local`:

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo vim /etc/fail2ban/jail.local
```

Add (or edit) these two sections — also in [config/fail2ban/jail.local-sections.conf](config/fail2ban/jail.local-sections.conf):

```ini
[sshd]
enabled = true
port = 24816
backend = systemd

[proxmox]
enabled = true
port = https,http,8006
filter = proxmox
backend = systemd
maxretry = 3
findtime = 2d
bantime = 1h
```

- `port`: the ports to block for the banned IP. For SSH, your **new** port.
- `maxretry = 3`, `findtime = 2d`, `bantime = 1h`: **3 failed attempts within 2 days → blocked for 1 hour.**

Create the filter for the web UI — [config/fail2ban/filter.d/proxmox.conf](config/fail2ban/filter.d/proxmox.conf):

```bash
sudo vim /etc/fail2ban/filter.d/proxmox.conf
```

```ini
[Definition]
failregex = pvedaemon\[.*authentication failure; rhost=<HOST> user=.* msg=.*
ignoreregex =
journalmatch = _SYSTEMD_UNIT=pvedaemon.service
```

Restart fail2ban:

```bash
sudo systemctl restart fail2ban
```

## Test it

Make one failed login on the web UI, then check that the filter finds it:

```bash
sudo fail2ban-regex systemd-journal /etc/fail2ban/filter.d/proxmox.conf
```

The output should show at least one `failregex` match.

Then test it for real **from another device** (I use my phone): 3 wrong passwords and the page stops loading.

```bash
sudo fail2ban-client status                 # active jails
sudo fail2ban-client get proxmox banned     # banned IPs in the proxmox jail
sudo fail2ban-client unban 192.168.1.23     # unban yourself (use the IP you got banned from)
```

> Don't test from the only computer you manage the server from — or keep the web UI Shell / server console at hand to unban yourself.

Sources: [Proxmox Wiki — Fail2ban](https://pve.proxmox.com/wiki/Fail2ban) · [Proxmox Forum — Fail2ban on Proxmox 9](https://forum.proxmox.com/threads/fail2ban-proxmox-9.175815/)
