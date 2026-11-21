# 07 — First images: VM vs container

## Download an ISO (for virtual machines)

`local (pve) → ISO Images → Download from URL` → paste the link to the ISO → **Query URL** → **Download**. Proxmox downloads it straight to the server.

Example: the Debian netinst ISO from [debian.org/distrib/netinst](https://www.debian.org/distrib/netinst) (copy the `amd64` link).

## Download a container template (for CTs)

`local (pve) → CT Templates → Templates` → pick **debian-13-standard** → **Download**.

From the shell:

```bash
pveam update                       # refresh the list of templates
pveam available --section system   # list the available OS templates
pveam download local debian-13-standard_<version>_amd64.tar.zst   # use the exact name from the list
```

## VM or CT?

Back to the house:

- A **virtual machine (VM)** is a **detached house**: its own full system, with **its own kernel**.
- A **container (CT)** is an **apartment**: it shares the foundation, **the server's kernel**.

| | VM | CT (LXC container) |
|---|---|---|
| Kernel | Its own | Shares the host's |
| Operating systems | Any (Linux, Windows, BSD…) | Linux only |
| Isolation | Strong | Lower |
| Resources | Heavier | Lighter |
| Boot time | Like a real PC | Seconds |

**My rule:**

- A **Linux service** → **container**.
- **Windows**, something that needs **its own kernel**, or something I **don't fully trust** → **VM**.
- **Docker** → **VM** too. Running application containers inside a VM is what Proxmox recommends for the best isolation.

Source: [Proxmox VE Admin Guide — Linux Container](https://pve.proxmox.com/pve-docs/chapter-pct.html)
