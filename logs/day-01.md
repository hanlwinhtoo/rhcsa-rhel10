# Day 01 — Lab Build

**Date:** 2026-10-02
**Objective(s):** Lab setup (prerequisite for all objectives)
**Time spent:** ~2.5 hours

## What I built
- Isolated Proxmox SDN network `rhcsanet` (10.10.10.0/24) with DHCP and SNAT, separate from my home LAN and CEH lab.
- servera (10.10.10.100) and serverb on RHEL 10.2, Server install, no GUI.
- `clean_install` snapshots on both VMs.
- SSH access from my laptop through the Proxmox host (`ssh -J`).

Step-by-step guide: [lab/lab-setup.md](../lab/lab-setup.md)

## Commands I used

| Command | What it does |
|---|---|
| `apt install -y dnsmasq` | install a package on Debian/Proxmox |
| `systemctl disable --now dnsmasq` | stop a service now and stop it starting at boot |
| `systemctl status <service>` | check if a service is running/enabled |
| `pwd` | print working directory |
| `ls` / `ls -a` | list files / include hidden dot-files |
| `sudo <command>` | run one command as root |
| `sudo poweroff` | shut down cleanly |
| `ping -c 4 <host>` | send 4 test packets |
| `curl -I <url>` | fetch only a web page's headers |
| `ssh -J <jump> <user>@<host>` | SSH through a jump host |
| `qm listsnapshot <vmid>` | list a Proxmox VM's snapshots |
| `exit` | close the shell / SSH session |
| Ctrl+C | cancel a running command |

## Problems I hit and how I fixed them

| Problem | Cause | Fix |
|---|---|---|
| RHEL 10 might not boot | Proxmox default CPU type is x86-64-v2 | Set CPU type to `host` |
| Couldn't find where to Apply SDN | Was on Node → System → Network | Datacenter → SDN → Apply |
| `yes` didn't appear when typing in PowerShell | Windows `ssh -J` doesn't echo the first prompt | It was accepted anyway ("Permanently added") |
| `ping redhat.com` showed 100% loss | Red Hat blocks ping | `curl -I` proved DNS and internet work |
| Home folder looked empty | Server install has no desktop folders; hidden files start with `.` | `ls -a` |

## Exam lessons
- ⚠️ `start` ≠ `enable`. Enable = a symlink in `multi-user.target.wants/`. Configurations must survive a reboot.
- Read all output before diagnosing; a failed ping alone doesn't mean a broken network.
- Test one layer at a time: IP number first, then name.
- Prompt ending in `$` = normal user, `#` = root.

## Offline help used
- `man systemctl`, `man ping`, `man ls` (press `q` to quit)

## Still to do
- [ ] Confirm serverb got 10.10.10.101 and can ping servera
- [ ] Isolation test from the CEH lab
- [ ] Objective 1 hands-on lab

## Next session
Objective 1 — access a shell prompt and issue commands with correct syntax.
