# 02 — The firewall: two switches

A fresh Proxmox install already has several services listening (the web UI, SSH and a few more). Check it yourself:

```bash
ss -tulpn   # TCP/UDP sockets that are listening, with the process that owns each one
```

And the Proxmox firewall comes **turned off**. The idea: close everything and open only what you use.

## Datacenter vs node: why you need both

Proxmox has a firewall at two levels:

| Level | Where | What it does | Default |
|---|---|---|---|
| **Datacenter** | `Datacenter → Firewall` (`/etc/pve/firewall/cluster.fw`) | The **main switch**. Its rules and policy apply to **every node** in the cluster. | **Off** |
| **Node** | `pve → Firewall` (`/etc/pve/nodes/<node>/host.fw`) | The firewall of **this server**, with its own options and rules. | **On**, but it does nothing while the Datacenter firewall is off |

Turning on only the node does nothing; turning on only the Datacenter while a node has its firewall off leaves that node unprotected. **Both must be on.**

## The 3 rules

I put them in the **Datacenter**, so they also apply to any server I add later. `Datacenter → Firewall → Add`:

| # | Direction | Action | Protocol | Dest. port | Comment |
|---|---|---|---|---|---|
| 1 | in | ACCEPT | tcp | 8006 | Web UI |
| 2 | in | ACCEPT | tcp | 22 | SSH (changed to the new port in [04](04-sshd-config.md)) |
| 3 | in | DROP | | | Everything else |

Rules are read **top to bottom** and the **first match wins**, so the order matters. Check the order in the list and drag the rules if needed.

On my home network I leave them open to **any source**. On a server exposed to the internet, limit rules 1 and 2 to a list of trusted IPs (an **IPSet** in `Datacenter → Firewall → IPSet`, used as the rule's source).

Example: [config/cluster.fw.example](config/cluster.fw.example).

## Turn it on without locking yourself out

1. **Keep an SSH session open** before turning it on, in case something goes wrong.
2. `Datacenter → Firewall → Options` → **Firewall: Yes**, and check **Input Policy: DROP** (anything that comes in and isn't in the rules is dropped).
3. `pve → Firewall → Options` → check that **Firewall: Yes** (it already comes on).

Check the status from the shell:

```bash
pve-firewall status
```

### Good to know

- With the input policy on DROP, Proxmox still allows a few things from your **local management network** on its own (an anti-lockout list that includes ports 8006 and 22). A **new SSH port is not on that list**, so it needs its own rule (see [04](04-sshd-config.md)).
- Rescue: if you lock yourself out, use the web UI **Shell** or the server console and run `pve-firewall stop` until you fix the rules.

Source: [Proxmox VE Admin Guide — Firewall](https://pve.proxmox.com/pve-docs/chapter-pve-firewall.html) · [Proxmox Wiki — Firewall](https://pve.proxmox.com/wiki/Firewall)
