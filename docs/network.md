# Network

## VLANs

Subnets are proposed. Before using them, confirm the home router's LAN doesn't overlap (some ISP gateways hand out 10.0.0.0/24).

| VLAN | Name | Subnet | Gateway | Members |
|---|---|---|---|---|
| 10 | WAN | DHCP from home router | home router | OPNsense WAN |
| 20 | Lab devices | 10.20.20.0/24 | 10.20.20.1 | Opal AP, ThinkPad, lab Wi-Fi clients |
| 30 | Targets | 10.20.30.0/24 | 10.20.30.1 | Ubuntu target, Windows target |
| 99 | Management | 10.20.99.0/24 | 10.20.99.1 | Proxmox host, Wazuh, ntfy, switch management |

## Switch ports (TL-SG108E)

| Port | Mode | VLAN(s) | Connected to |
|---|---|---|---|
| 1 | access | 10 | Home router |
| 2 | trunk | 10, 20, 30, 99 tagged | 7050 |
| 3 | access | 20 | Opal (AP mode) |
| 8 | access | 99 | Break glass: plug a laptop in here if OPNsense or Tailscale is down |
| 4 to 7 | unused | none | Remove from VLAN 1 so a random cable gets nothing |

The break glass port is there because the Proxmox web UI lives on VLAN 99. If OPNsense is broken, a laptop on port 8 with a static 10.20.99.x address can still reach the hypervisor on the same layer 2 segment. Keep a monitor and keyboard handy anyway for the first VLAN aware bridge change; that is the step most likely to lock you out.

To verify: TP-Link Easy Smart switches have a history of weak management protocol security. Check current firmware and advisories, restrict management to VLAN 99 if the firmware allows it, and record the result in a decision note.

## Firewall intent

Default deny between VLANs. Rules below are the allow list.

| From | To | Allow | Why |
|---|---|---|---|
| Lab devices (20) | Internet | any | Updates, browsing from Kali |
| Lab devices (20) | Targets (30) | any | Kali attacks targets |
| Targets (30) | Wazuh | 1514/tcp, 1515/tcp | Agent events and enrollment |
| Targets (30) | Internet | 80, 443 | Updates. Tighten later and alert on anything else |
| Targets (30) | Management (99) | nothing except Wazuh ports above | A popped target must not reach Proxmox |
| Lab devices (20) | Management (99) | Wazuh 1514/1515 for the ThinkPad agent | Agent only. Admin goes over Tailscale |
| Tailnet (admin tag) | Management (99) | 22, 443, 8006 | SSH, Wazuh dashboard, Proxmox UI |
| Management (99) | Internet | 80, 443 | Updates |

## Tailscale

- OPNsense runs the subnet router and advertises 10.20.99.0/24 only.
- ACL tags: `tag:admin` (Kali, phone), `tag:siem` (Wazuh, ntfy), `tag:target` (anything on VLAN 30 that ever joins the tailnet).
- The policy file goes in `infra/tailscale/acl.hujson` with the tailnet name and any device names that identify me removed.

## SSH

OpenSSH on every Linux host, not Tailscale SSH, so sshd logs reach Wazuh. Keys only: `PasswordAuthentication no`, `PermitRootLogin no`, `KbdInteractiveAuthentication no`. The hardened config goes in `infra/ssh/`. RDP only to the Windows target, never port forwarded.

## Open questions

- **Gaming PC agent path.** If the gaming PC stays on the household network, it sits outside OPNsense and can't reach Wazuh on VLAN 99 without either a port forward on OPNsense WAN (no) or joining the tailnet with a `tag:monitored` ACL that allows only 1514/1515 to Wazuh. Tailnet looks cleaner. Decide in Weekend 2.
- **Wazuh address for agents.** Agents on VLAN 20 and 30 point at 10.20.99.x. The gaming PC over Tailscale would point at the same address through the subnet route.
