# NNN: Detection name

| | |
|---|---|
| **Rule ID(s)** | 1NN00 (see `wazuh/rules/local_rules.xml`) |
| **ATT&CK** | Txxxx.xxx, technique name |
| **Log source** | e.g. sshd via Wazuh agent, OPNsense DHCP syslog, Sysmon Event ID 1 |
| **Severity** | Wazuh level N |
| **Alerts to phone** | yes / no |
| **Status** | planned / testing / tuned / retired |

## What it detects

Two or three sentences. The behavior, why an attacker does it, and why it matters in this lab.

## Logic

Plain English first, then point to the rule by ID. Do not paste the XML here; `local_rules.xml` is the single source of truth.

> Fires when ... within ... seconds from the same ...

## Sample triggering log

Sanitized. Real MACs replaced with 00:00:5e:00:53:xx, no public IPs.

```
paste the raw log line(s) here, or reference sample.log in this folder
```

## How to reproduce

Exact commands from Kali (or wherever) that make the rule fire, so anyone can confirm it works.

```
command here
```

Expected result: alert at level N in the dashboard, and a phone notification if enabled.

## Known false positives

- Source and how to recognize it

## Tuning

Summary of changes, with links to rows in [tuning-log.md](../../tuning/tuning-log.md).

## Blind spots

What this detection will miss, and why. Example: tailnet SSH sessions show OPNsense as the source (see decision 0004).

## Response

First three things an analyst should check when this fires.

1.
2.
3.
