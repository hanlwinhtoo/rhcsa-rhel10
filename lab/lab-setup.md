# Lab Setup — Proxmox + Two RHEL 10 VMs

A rebuild-from-scratch guide for my RHCSA lab. Each setting includes **why**, and the traps are marked ⚠️.

## 1. Network plan

| Network | Range | Purpose |
|---|---|---|
| `vmbr0` | 192.168.1.0/24 (host .200) | Home LAN, Proxmox web UI |
| `vmbr1` | 10.10.1.0/24 (host .2) | Separate CEH lab — not touched |
| `rhcsanet` | 10.10.10.0/24 (gateway .1) | RHCSA lab |
| 10.10.10.2 – .99 | reserved | Static IPs later (Phase 7) |
| 10.10.10.100 – .199 | DHCP pool | Addresses handed out automatically |

Analogy: the Proxmox host is a building. `rhcsanet` is a private floor with its own room numbers (IPs), a receptionist handing them out (DHCP via dnsmasq), and one shared exit to the street (SNAT).

## 2. Build the isolated network (Proxmox SDN)

### A. Install dnsmasq (DHCP for SDN)
On the Proxmox host shell:
```bash
apt update && apt install -y dnsmasq
systemctl disable --now dnsmasq
```
- `apt` = Debian's package installer (Proxmox is Debian-based; RHEL uses `dnf`).
- The default dnsmasq copy grabs DNS/DHCP ports on every interface. Proxmox starts its own copy per zone, so the default one must be off or the zone's copy fails.

Verify:
```bash
systemctl status dnsmasq     # expect: disabled, inactive (dead)
```

> **Exam lesson:** `enable` creates a symlink in `/etc/systemd/system/multi-user.target.wants/` so a service starts at boot. `start` only runs it now. Start without enable = gone after reboot = zero marks.

### B. Zone — Datacenter → SDN → Zones → Add → Simple
- ID `rhcsa`, IPAM `pve`, **Automatic DHCP ☑**, DNS fields blank.

### C. VNet — SDN → VNets → Create
- Name `rhcsanet` (max 8 chars), Zone `rhcsa`.
- ⚠️ **Isolate Ports ☐** — leave off, or servera and serverb can't talk (SSH, NFS, autofs need that).

### D. Subnet — select `rhcsanet` → Subnets → Create
- Subnet `10.10.10.0/24` (must end in **.0**), Gateway `10.10.10.1`, **SNAT ☑**.
- ⚠️ **DHCP Ranges tab** → `10.10.10.100` to `10.10.10.199` **before** clicking Create.

### E. Apply — Datacenter → SDN → Apply
Nothing exists until Apply runs. Status should show **available**.

> ⚠️ Node → System → Network is the host's own config. Don't create anything there, and don't click its "Apply Configuration". `rhcsanet` never appears in that list — that's normal.

### How isolated is it?
Separate layer 2 (own broadcasts, own DHCP, no IP clashes), but the host routes between its networks because SNAT needs IP forwarding. The CEH lab may still reach the RHCSA VMs. **To do:** ping servera from the CEH attack box; if it answers, add a Proxmox firewall rule blocking 10.10.1.0/24 ↔ 10.10.10.0/24.

## 3. Create a VM

| Tab | servera | serverb | Why |
|---|---|---|---|
| General | ID 201, `servera` | ID 202, `serverb` | HA off, Start at boot off |
| OS | RHEL 10.2 DVD ISO, Linux, newest kernel | same | |
| Disks | 40 GB + 2 × 10 GB, **Discard ☑** | 40 GB + 1 × 10 GB, **Discard ☑** | Discard returns freed space to the thin pool |
| CPU | 2 cores, **Type `host`** | same | ⚠️ RHEL 10 needs x86-64-v3; default type is v2 and won't boot |
| Memory | 4096 MB | 4096 MB | |
| Network | **Bridge `rhcsanet`**, VirtIO | same | ⚠️ Default `vmbr0` puts it on the home LAN |

> ⚠️ "vCPU Architecture" (General tab) is **not** CPU Type. Leave it default.

## 4. Install RHEL 10.2

1. **Installation Destination** — ⚠️ tick **only the 40 GiB disk**. Automatic. No encryption (a boot passphrase blocks reboot checks).
2. **Software Selection** — **Server** (no GUI; the exam is command line).
3. **Network & Host Name** — card ON (expect 10.10.10.1xx). Hostname `servera.rhcsa.internal` / `serverb.rhcsa.internal`, ⚠️ click **Apply** and confirm "Current host name" changed.
4. **Time & Date** — Asia/Singapore, Network Time ON.
5. **Root Password** — set, ☑ Allow root SSH login with password.
6. **Create User** — `admin`, ☑ Make this user administrator (wheel group → sudo; it is not root).
7. **Connect to Red Hat** — skip; registration is a Phase 3 command-line task.
8. Begin Installation → Reboot.

## 5. Verify

```bash
hostname                          # servera.rhcsa.internal
ping -c 4 10.10.10.1              # gateway (Proxmox host)
ping -c 4 8.8.8.8                 # internet path — tests SNAT
curl -I https://www.redhat.com    # DNS + web — any HTTP/... line = success
```
- 8.8.8.8 is Google Public DNS. Pinging a **number** tests the network path without DNS; pinging a **name** also tests DNS.
- ⚠️ `redhat.com` ignores ping (100% loss) but the name still resolved to an IP → DNS works. A failed ping doesn't prove the network is broken. Read every line.

## 6. Snapshots

```bash
sudo poweroff                     # shut down cleanly first
```
- Proxmox → VM → Snapshots → Take Snapshot → `clean_install` (no hyphens allowed).
- Check: `qm listsnapshot 201` on the Proxmox shell.
- Rollback discards everything after the snapshot. A snapshot lives on the same disk — it is an undo button, not a backup.

## 7. Access from my Windows laptop

The laptop can't route to 10.10.10.x, so jump through the Proxmox host:
```powershell
ssh -J root@192.168.1.200 admin@10.10.10.100
```
- Two passwords: Proxmox root, then servera's admin.
- First connect asks to trust each host's **fingerprint** — type the full word `yes`. Stored in `C:\Users\<me>\.ssh\known_hosts`.
- Windows quirk: with `-J`, the first `yes` may not echo on screen. If stuck, `ssh root@192.168.1.200` once, `exit`, then retry.
- Optional later: Tailscale subnet route for 10.10.10.0/24 to reach the lab from outside home.
