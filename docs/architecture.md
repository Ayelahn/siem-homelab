# Architecture

## Physical and logical topology

The home router stays the household gateway. Everything in the lab sits behind OPNsense, so if the 7050 goes down only the lab loses connectivity.

```mermaid
flowchart TB
    internet((Internet))
    home[Home router<br/>household network]
    sw[TL-SG108E switch]
    opal[GL.iNet Opal<br/>AP mode, lab SSID]
    kali[ThinkPad<br/>Kali]
    breakglass[Break glass port<br/>VLAN 99 access]

    internet --- home
    home -->|VLAN 10 access port| sw
    sw -->|VLAN 20 access port| opal
    opal -.->|Wi-Fi| kali
    sw --- breakglass

    subgraph pve[OptiPlex 7050 Micro, Proxmox VE 9, single NIC, VLAN aware bridge]
        opn[OPNsense<br/>gateway, firewall, Suricata<br/>DNS and DHCP, Tailscale subnet router]
        wazuh[Wazuh all in one<br/>VLAN 99]
        ntfy[ntfy LXC<br/>VLAN 99]
        ubu[Ubuntu target<br/>VLAN 30]
        win[Windows Server + Sysmon<br/>VLAN 30]
    end

    sw ==>|trunk: 10, 20, 30, 99| opn
    opn --- wazuh
    opn --- ntfy
    opn --- ubu
    opn --- win

    phone[Phone<br/>Tailscale client] -.->|tailnet| opn
```

## Log and alert flow

```mermaid
flowchart LR
    ubu[Ubuntu target agent] -->|1514/tcp| wazuh
    win[Windows target agent<br/>Security + Sysmon] -->|1514/tcp| wazuh
    kali[ThinkPad agent] -->|1514/tcp| wazuh
    gpc[Gaming PC agent] -.->|path TBD, see network.md| wazuh
    opn[OPNsense] -->|syslog: filterlog, DHCP, DNS| wazuh
    suri[Suricata EVE JSON] -->|syslog| wazuh
    wazuh[Wazuh manager<br/>rules + decoders] -->|custom-ntfy integration| ntfy[ntfy]
    ntfy -->|push over tailnet| phone[Phone]
    wazuh --> dash[Wazuh dashboard]
```

## Guest sizing

| Guest | vCPU | RAM | Disk |
|---|---|---|---|
| OPNsense | 2 | 4 GB | 32 GB |
| Wazuh all in one (Ubuntu 24.04) | 4 | 10 GB | 200 GB |
| Ubuntu target | 2 | 2 GB | 32 GB |
| Windows Server eval + Sysmon | 2 | 6 GB | 64 GB |
| ntfy LXC | 1 | 512 MB | 4 GB |

Total: 11 vCPU on a 4 core host (overcommitted on purpose, the targets sit idle most of the time) and about 22.5 GB of 32 GB RAM, leaving headroom for Proxmox itself.

## Known weak points

- The 7050 is a single point of failure for the lab. Household traffic is unaffected.
- Single NIC means WAN and LAN share one physical link. Fine at home lab volumes.
- Tailscale subnet router SNAT hides the real source of SSH sessions arriving over the tailnet. See [decisions/0004](decisions/0004-tailscale-snat-blind-spot.md).
