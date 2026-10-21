# 2026-10-21: Plan locked, repo created

## Done

- Scoped the project to SIEM plus Tailscale VPN. No Plex, game servers, or AI triage until phase 1 is published.
- Picked Wazuh ([0001](../decisions/0001-wazuh-as-siem.md)) and a wired, router on a stick topology ([0002](../decisions/0002-wired-uplink-router-on-a-stick.md)).
- Created this repo with a gitleaks pre-commit hook before any configs exist.

## Problems

- The Opal can't run Tailscale (stuck on Siflower OpenWrt 18.06), which forced the switch purchase.
- Tailscale's fix for subnet router SNAT is Linux only, so SSH source IPs over the tailnet will be hidden behind OPNsense. Documented in [0004](../decisions/0004-tailscale-snat-blind-spot.md).

## Next

- Order the TL-SG108E and three ethernet cables.
- Weekend 1: reimage Proxmox, VLANs, OPNsense, Opal to AP mode, Tailscale, SSH hardening.

<!--
Format for future entries. One file per work session, named YYYY-MM-DD-short-topic.md.
Keep it short. The "Problems" section is the part that turns into interview answers,
so write down the actual error, what you tried, and what fixed it.
-->
