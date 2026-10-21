# 0001: Wazuh as the SIEM

**Date:** 2026-10-21
**Status:** accepted

## Context

The lab needs a SIEM that alerts for free, fits in about 10 GB of RAM next to four other guests, and covers host telemetry as well as network logs.

## Options considered

- **Wazuh.** Free alerting, file integrity monitoring, security configuration assessment, vulnerability detection, and agents for Linux and Windows. All in one install fits the RAM budget.
- **Splunk Free.** The SPL skills are the most requested on SOC job postings, but the free license drops alerting and caps ingest. A SIEM that can't alert defeats the ntfy part of the project.
- **Security Onion.** Strong network visibility, but its standalone footprint is larger than what the 7050 can spare after OPNsense and two targets.
- **Elastic stack by hand.** Flexible, but more time spent on plumbing than on detections.

## Decision

Wazuh 4.x stable, all in one, on Ubuntu 24.04.

## Consequences

- Detection logic is written as Wazuh XML rules, not SPL or KQL. To close that resume gap separately: SPL through Splunk's Boss of the SOC datasets, KQL through Microsoft Learn.
- Wazuh's indexer is the RAM hog. If the 10 GB guest is unstable, the first fix is tuning indexer JVM heap, not adding guests.
