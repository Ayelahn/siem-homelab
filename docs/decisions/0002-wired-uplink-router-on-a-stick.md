# 0002: Wired uplink with OPNsense as router on a stick

**Date:** 2026-10-21
**Status:** accepted

## Context

The GL.iNet Opal (GL-SFT1200) is stuck on Siflower OpenWrt 18.06 and cannot run Tailscale, so it can't be the lab gateway or the remote access point. The 7050 has a single NIC.

## Decision

Buy a TP-Link TL-SG108E. The home router feeds the switch on an access port in VLAN 10. One trunk carries VLANs 10, 20, 30, and 99 to the 7050, where a VLAN aware Proxmox bridge hands them to OPNsense. OPNsense routes and filters between them. The Opal drops to access point mode for the lab SSID.

## Consequences

- OPNsense sees every lab flow, which gives Suricata and the DHCP and DNS logs full coverage of the lab.
- The 7050 becomes a single point of failure for the lab only. The household network stays on the home router.
- Double NAT (home router, then OPNsense). Acceptable because nothing is port forwarded and remote access is outbound Tailscale.
- Changing the Proxmox bridge to VLAN aware is the riskiest step of Weekend 1. Have a monitor and keyboard on the 7050 before touching it, and keep switch port 8 as a VLAN 99 break glass port.
