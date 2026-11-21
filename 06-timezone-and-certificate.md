# 06 — Time zone and certificate

## Time zone

If the time zone is wrong, the timestamps in your logs (and in fail2ban) won't add up.

- **Web UI:** `pve → System → Time → Edit` → choose your time zone (mine: `America/Bogota`).
- **Shell:**
  ```bash
  timedatectl                                  # show the current time zone
  sudo timedatectl set-timezone America/Bogota
  ```

## Your own SSL certificate

The "connection not secure" warning shows up because Proxmox creates its own self-signed certificate at install time.

I have a public **wildcard** certificate for `*.itslucho.com`, so I upload it:

1. `pve → System → Certificates → Upload Custom Certificate`.
2. **Private key:** paste the key (it must **not** have a password).
3. **Certificate chain:** paste the **full chain** (your certificate + the intermediate certificates).
4. Click **Upload**. The web interface restarts; reload the page and the warning is gone.

From the shell, the same thing:

```bash
sudo pvenode cert set fullchain.pem privkey.pem --force --restart
```

Proxmox stores them as `/etc/pve/local/pveproxy-ssl.pem` and `/etc/pve/local/pveproxy-ssl.key`.

> **Never show or share the private key.** When the certificate expires, upload the new one the same way.

### Don't have a certificate?

- **ACME (built in, free):** Proxmox can get and **renew automatically** a Let's Encrypt certificate. `Datacenter → ACME` (add an account and, if your server isn't reachable from the internet, a **DNS** plugin for your DNS provider), then `pve → System → Certificates → ACME → Add` your domain → **Order Certificate Now**.
- **certbot:** get the certificate with [certbot](https://certbot.eff.org/) and load it with `pvenode cert set` as shown above.

## The name `pve.itslucho.com`

For my computer to find the server by its name, for now I resolve it in the **hosts** file of my Mac:

```bash
sudo nano /etc/hosts
# add a line like:
# 192.168.1.50   pve.itslucho.com
```

In an upcoming part of the series I move this to a **local DNS** for the whole network.

Source: [Proxmox Wiki — Certificate Management](https://pve.proxmox.com/wiki/Certificate_Management)
