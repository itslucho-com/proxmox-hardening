# proxmox-hardening

Commands, settings and notes from the **Homelab From Scratch** series by [It's Lucho](https://www.youtube.com/@itslucho_com).

## Part 2 — Secure Your Proxmox Server: Firewall, SSH and Fail2ban

▶️ Video: https://youtu.be/DPPcs6L9xIU · 📖 Written guide: https://www.itslucho.com/writing/secure-your-proxmox-server/

A fresh Proxmox install is like a house with every door and window open: anyone on your network can try to log in as `root` with just a password. In this part I close everything and leave a single door, with a key nobody can copy. Everything shown in the video is here, so you can do the same on your own Proxmox.

| File | What's inside |
|---|---|
| [01-repositories.md](01-repositories.md) | Switch from the enterprise repository to the free **No-Subscription** one and update the server. |
| [02-firewall.md](02-firewall.md) | The **Datacenter** firewall vs the **node** firewall (and why you need both), the 3 rules and how not to lock yourself out. |
| [03-user-and-ssh-key.md](03-user-and-ssh-key.md) | Your own user with `sudo` and an **SSH key** (on a YubiKey, FIDO2 or a normal key file). |
| [04-sshd-config.md](04-sshd-config.md) | **Every `sshd_config` setting explained**, how to pick a new SSH port and the safe order to apply it all. |
| [05-fail2ban.md](05-fail2ban.md) | **Fail2ban** for SSH and the Proxmox web UI (port 8006) on Proxmox VE 9. |
| [06-timezone-and-certificate.md](06-timezone-and-certificate.md) | Time zone, uploading your own SSL certificate, and ACME / certbot as alternatives. |
| [07-images-vm-vs-ct.md](07-images-vm-vs-ct.md) | Download your first ISO and container template, and when to use a VM or a container (CT). |
| [config/](config/) | Ready-to-copy config files: `sshd_config` settings, firewall example and fail2ban jail + filter. |

## The safe order (read this first)

1. **Repositories and updates** — you need them to install `sudo` and `fail2ban`.
2. **Firewall** — keep an SSH session open while you turn it on.
3. **Your user + SSH key** — test that the key works **before** you turn off passwords.
4. **SSH settings + new port** — open the new port in the firewall **first**, check the config, restart SSH and test from a **second** terminal before closing the first one.
5. **Fail2ban.**
6. **Time zone and certificate.**

### If you lock yourself out

You can always get back in from the **Shell** in the Proxmox web UI (`pve` → **Shell**, as `root@pam`) or from the server's physical console / iLO. From there, undo the last change and restart the service. If the firewall is the problem, `pve-firewall stop` turns it off until you fix the rules.

## My setup

| | |
|---|---|
| Server | HP ProLiant ML310e Gen8 v2 (2013) · Xeon E3-1240 v3 · 24 GB DDR3 ECC |
| Hypervisor | Proxmox VE 9.2 (Debian 13 "Trixie") — single node |
| SSH key | YubiKey 5C NFC (OpenPGP authentication subkey via `gpg-agent`), with the same key on a second YubiKey as a backup |
| Network | Home network (LAN only). The web UI and SSH are **not** exposed to the internet. |

> The IPs and the SSH port in these files are **examples** (`192.168.1.x`, port `24816`). Use your own.

## Homelab From Scratch

1. **Install Proxmox on an Old Server** — [video](https://www.youtube.com/watch?v=fsqaijAtEkE) · [repo: proxmox-setup](https://github.com/itslucho-com/proxmox-setup)
2. **Secure Your Proxmox Server: Firewall, SSH and Fail2ban** — this part
3. _Coming next…_

## License

[MIT](LICENSE) © 2026 Luis Jaramillo (It's Lucho)

---

🌐 [itslucho.com](https://www.itslucho.com) · ▶️ [YouTube](https://www.youtube.com/@itslucho_com) · 💼 [LinkedIn](https://www.linkedin.com/in/itslucho-com/)
