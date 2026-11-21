# 01 — Free repository and updates

Out of the box, Proxmox VE uses the **enterprise** repository, which needs a paid subscription. Without one, `apt update` fails on that repository (`401 Unauthorized`) and you don't get Proxmox updates.

For a homelab, use the free **No-Subscription** repository. Proxmox doesn't recommend it for production, but it's the right one for a lab.

## Option A — From the web UI (what I do in the video)

1. Go to **`pve` → Updates → Repositories**.
2. Select the **enterprise** repositories (`pve-enterprise` and, if it's there, the Ceph enterprise one) and click **Disable**.
3. Click **Add**, choose **No-Subscription** and click **Add**.

## Option B — From the shell

Proxmox VE 9 uses the deb822 format (`.sources` files) in `/etc/apt/sources.list.d/`.

Disable the enterprise repository by adding `Enabled: no` to it:

```bash
nano /etc/apt/sources.list.d/pve-enterprise.sources
```

```
Types: deb
URIs: https://enterprise.proxmox.com/debian/pve
Suites: trixie
Components: pve-enterprise
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
Enabled: no
```

Do the same with the enterprise entry in `/etc/apt/sources.list.d/ceph.sources` if you have one.

Add the No-Subscription repository in `/etc/apt/sources.list.d/proxmox.sources`:

```
Types: deb
URIs: http://download.proxmox.com/debian/pve
Suites: trixie
Components: pve-no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
```

## Update

```bash
apt update          # download the list of available packages
apt full-upgrade    # install every update
```

Use `full-upgrade` (or `dist-upgrade`), **never plain `apt upgrade`**, on Proxmox: some updates need to add or remove packages.

If the kernel was updated, reboot:

```bash
reboot
```

## "No valid subscription"

The pop-up at login **keeps showing** even with the free repository. It's only a reminder: Proxmox VE is free and open source, and the subscription is for enterprise support and the enterprise repository. Leave it be.

Source: [Proxmox Wiki — Package Repositories](https://pve.proxmox.com/wiki/Package_Repositories)
