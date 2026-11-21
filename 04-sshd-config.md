# 04 — Locking down SSH and changing the port

With your key working (see [03](03-user-and-ssh-key.md)), close everything else.

```bash
sudo vim /etc/ssh/sshd_config     # or nano, if you prefer
```

## Every setting, explained

| Setting | Value | What it does |
|---|---|---|
| `PermitRootLogin` | `no` | Nobody can log in directly as `root`. You log in as your user and use `sudo`. |
| `PasswordAuthentication` | `no` | No passwords: only SSH keys. Bots guessing passwords get nowhere. |
| `PermitEmptyPasswords` | `no` | Accounts with an empty password can never log in. The default is already `no`, but it's good to be explicit. |
| `AllowUsers` | `itslucho` | **Only** the users listed here can log in over SSH. Everyone else is rejected, even with a valid key. |
| `ClientAliveInterval` | `100` | Every 100 seconds of silence, the server checks if your client is still there. |
| `ClientAliveCountMax` | `3` | After 3 checks with no answer (100 × 3 = 300 s, **about 5 minutes**), the session is closed. Hung sessions don't stay open. |
| `X11Forwarding` | `no` | Turns off forwarding of graphical (X11) apps. A server doesn't need it, and it's one less thing exposed. |
| `Port` | `24816` (example) | The new SSH port. See "Picking a port" below. |

Ready-to-copy block: [config/sshd_config-hardening.conf](config/sshd_config-hardening.conf).

### What about `Protocol 2`?

Many old guides still add `Protocol 2`. OpenSSH removed the old SSH-1 protocol years ago (server support in 7.4, the option itself in 7.6), so that line does nothing today. I don't include it.

### Debian detail: the first value wins

On Debian (and Proxmox), `sshd_config` starts with `Include /etc/ssh/sshd_config.d/*.conf`, and for each setting **the first value found wins**. If a file in that folder sets `PasswordAuthentication yes`, it beats what you write lower in `sshd_config`. Always check the **effective** values:

```bash
sudo sshd -T | grep -Ei '^(port|permitrootlogin|passwordauthentication|permitemptypasswords|allowusers|clientaliveinterval|clientalivecountmax|x11forwarding) '
```

## Picking a port

Bots scan port 22 all day long. Changing it doesn't make you invincible (the real protection is the key), but it removes most of the noise.

- Use a **high port between 1024 and 32767**.
  - Below 1024 are the well-known system ports.
  - From 32768 up, Linux uses ports for its own outgoing connections (`net.ipv4.ip_local_port_range` is `32768 60999` by default).
- Make sure **no other service uses it**: `ss -tulpn | grep :24816` should return nothing.
- Avoid the obvious alternatives (`2222`, `2022`, `22222`…): bots try those too.

## The safe order (so you don't lock yourself out)

1. **Open the new port in the firewall first**: `Datacenter → Firewall → Add` → in, ACCEPT, tcp, dest. port `24816`, **above the DROP rule**.
2. **Check the config** for errors (no output means it's fine):
   ```bash
   sudo sshd -t
   ```
3. **Restart SSH**:
   ```bash
   sudo systemctl restart ssh
   ```
4. **Without closing your current session**, test from a **second terminal**:
   ```bash
   ssh -p 24816 itslucho@pve.itslucho.com   # your user gets in
   ssh -p 24816 root@pve.itslucho.com       # root: Permission denied
   ```
5. Only now, **delete the firewall rule for port 22**.

Tip: add the port to `~/.ssh/config` on your computer so you don't have to type `-p` every time:

```
Host pve
    HostName pve.itslucho.com
    User itslucho
    Port 24816
```

### Clusters

Proxmox clusters use SSH as `root` between nodes. These settings are for a **single node**. On a cluster, read the Proxmox docs before blocking root over SSH.

Sources: [sshd_config(5) — Debian](https://manpages.debian.org/testing/openssh-server/sshd_config.5.en.html) · [OpenSSH 7.6 release notes](https://www.openssh.com/txt/release-7.6) · [IANA port numbers](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml)
