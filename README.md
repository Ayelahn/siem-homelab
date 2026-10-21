# siem-homelab

A Wazuh SIEM and Tailscale remote access lab on a single Proxmox box, built to practice detection engineering, alert tuning, and incident response end to end.

**Status:** building. Started October 2026, target publish February 2027. Progress is in [docs/build-log](docs/build-log).

## Architecture

One Dell OptiPlex 7050 Micro runs Proxmox VE as a router on a stick. OPNsense is the lab gateway and IDS, Wazuh collects and alerts, and alerts reach my phone through a self hosted ntfy. Remote admin goes over Tailscale into the management VLAN only.

Full diagram and data flow: [docs/architecture.md](docs/architecture.md). VLANs, addressing, and firewall intent: [docs/network.md](docs/network.md).

| Component | Role |
|---|---|
| Proxmox VE 9 | Hypervisor on the 7050 (i5-7500T, 32 GB, 1 TB NVMe) |
| OPNsense | Gateway, firewall, Suricata IDS, DNS and DHCP logging, Tailscale subnet router |
| Wazuh 4.x (all in one) | SIEM: log analysis, FIM, SCA, vulnerability detection, alerting |
| Ubuntu 24.04 target | Linux target with Wazuh agent |
| Windows Server eval + Sysmon | Windows target with Wazuh agent |
| ntfy | Phone push alerts |
| Kali on a ThinkPad | Attacker and admin console |
| TP-Link TL-SG108E | 802.1Q switch, VLAN trunk to the 7050 |
| GL.iNet Opal | Access point for the lab SSID |

## Done criteria (phase 1)

- [ ] Architecture diagram
- [ ] `wazuh/rules/local_rules.xml` with custom rules
- [ ] Five detection writeups in [detections/](detections)
- [ ] Tuning log with before and after alert counts: [tuning/tuning-log.md](tuning/tuning-log.md)
- [ ] One incident report from a self run attack
- [ ] "New device on network" detection
- [ ] Phone alerts via ntfy
- [ ] Network baseline report from a week of DNS data

## Detections

| ID | Detection | ATT&CK | Source | Status |
|---|---|---|---|---|
| 001 | SSH brute force | T1110.001 | sshd via Wazuh agent | planned |
| 002 | New device on network | T1200 | OPNsense DHCP logs | planned |
| 003 | New SSH authorized key | T1098.004 | Wazuh FIM | planned |
| 004 | sudoers modification | T1548.003 | Wazuh FIM | planned |
| 005 | Windows failed logon burst (4625) | T1110 | Windows Security log | planned |
| 006 | Encoded PowerShell execution | T1059.001 | Sysmon Event ID 1 | planned |
| 007 | Suricata high severity escalation | varies | OPNsense Suricata EVE | planned |

Custom rule IDs live in the 100000 to 119999 range, one block of 100 per detection (001 uses 100100 to 100199, and so on).

## Repo layout

```
docs/          architecture, network, decision records, build log
infra/         sanitized configs: Proxmox, OPNsense, Tailscale ACL, sshd
wazuh/         rules, decoders, agent group configs, ntfy integration
detections/    one folder per detection: writeup + sample log
tuning/        tuning log
baseline/      network baseline report and the queries behind it
incidents/     incident reports
```

Folders appear when they get their first real file.

## Out of scope until phase 1 is published

Local AI alert triage on the gaming PC, Suricata in IPS mode, Zeek, and anything that isn't SIEM or VPN. Ideas go in an issue, not in the build.

## What never gets committed

Raw OPNsense `config.xml`, Tailscale auth or API keys, Wazuh credentials (including `wazuh-install-files.tar`), the ntfy topic or token, my public IP or tailnet name, unredacted MAC addresses, and packet captures. Secrets live in a gitignored `.env`; see [.env.example](.env.example). A gitleaks pre-commit hook enforces the rest.

To set up the hook after cloning:

```
pip install pre-commit
pre-commit install
```
