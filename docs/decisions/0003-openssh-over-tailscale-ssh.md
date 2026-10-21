# 0003: OpenSSH instead of Tailscale SSH

**Date:** 2026-10-21
**Status:** accepted

## Context

Tailscale SSH is convenient: no key distribution, and auth rides on tailnet identity. But sessions are handled by tailscaled rather than sshd, so the authentication events Wazuh decodes out of the box never appear in the host's auth log.

## Decision

Use OpenSSH on every Linux host, reached over the Tailscale subnet route. Keys only, no root login, no password or keyboard interactive auth.

## Consequences

- sshd auth events flow into Wazuh, which detection 001 (SSH brute force) and the incident report depend on.
- Key management is manual. Record which device holds which key by fingerprint in `infra/ssh/`, not the keys themselves, so the repo doesn't publish hostnames from key comments.
- Source IPs of tailnet sessions are affected by subnet router SNAT. See [0004](0004-tailscale-snat-blind-spot.md).
