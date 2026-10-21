# 0004: Tailscale subnet router SNAT hides SSH source addresses

**Date:** 2026-10-21
**Status:** proposed

## Context

A Tailscale subnet router masquerades traffic from tailnet clients by default. Every SSH session that arrives over the tailnet shows OPNsense's VLAN 99 address as the source in sshd logs, not the 100.x address of the device that connected. Wazuh can't tell the phone from Kali from a stolen laptop.

Tailscale's documented fix, `--snat-subnet-routes=false`, is marked Linux only. OPNsense runs FreeBSD, and neither Tailscale's OPNsense guide nor the os-tailscale plugin mentions a way to turn masquerading off. Treat it as unavailable until tested.

## Options

1. **Document the blind spot.** Note it in the architecture doc and in any detection that uses SSH source IPs. Zero added work.
2. **Move the subnet router to a small Linux LXC on VLAN 99** with `--snat-subnet-routes=false`, and add a static route on OPNsense for 100.64.0.0/10 pointing at that LXC. Real source IPs reach sshd. Costs one more guest and one more thing to break.
3. **Run Tailscale directly on the hosts that matter** (Wazuh, Proxmox) instead of reaching them through the subnet route. Sessions arrive from real 100.x addresses, but the "management VLAN only" boundary becomes a set of per host ACLs.

## Proposed decision

Option 1 for phase 1. The incident report attack runs from Kali on VLAN 20, not over the tailnet, so SNAT doesn't touch the main storyline. Revisit option 2 after publishing if a detection needs tailnet attribution.

## Consequences

- Any SSH alert with OPNsense's VLAN 99 address as the source means "came in over Tailscale, device unknown." Detection writeups that rely on source IP must say so.
- An attacker on the tailnet blends in with admin traffic in sshd logs. Before naming Tailscale's own logs as the compensating control, check which log types the free plan actually includes. Note the result in the incident report's detection gaps section.
