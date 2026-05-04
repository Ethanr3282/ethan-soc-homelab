# Home SOC Lab

A virtualized Security Operations Center built from scratch to learn defensive cybersecurity through hands-on detection engineering, threat hunting, and AI-assisted alert triage.

**Status:** 🚧 In progress — Phase 0 (setup)

## Goal

Build a working blue-team lab that mirrors what a real Security Operations Center does at a small scale: ingest logs from endpoints, detect simulated attacks with custom rules, hunt for threats, and triage alerts. Document every step so the project itself shows the workflow of a SOC analyst.

## Planned Architecture

Three virtual machines on a host-only network running on a MacBook Air (32 GB RAM):

- **Ubuntu Server** (192.168.56.10) — Wazuh SIEM (manager + indexer + dashboard)
- **Windows 10 Enterprise** (192.168.56.20) — victim endpoint with Sysmon + Wazuh agent
- **Kali Linux** (192.168.56.30) — attacker box for simulated attacks

## Roadmap

| Phase | Focus | Status |
|-------|-------|--------|
| 0 | Setup: VMs, networking, GitHub repo | 🚧 In progress |
| 1 | Wazuh + Sysmon foundation | ⏳ Planned |
| 2 | Detection engineering (custom rules + MITRE ATT&CK mapping) | ⏳ Planned |
| 3 | Threat hunting | ⏳ Planned |
| 4 | AI-powered alert triage (Anthropic API) | ⏳ Planned |
| 5 | Polish, blog writeup, demo video | ⏳ Planned |

## Tools

VirtualBox · Ubuntu Server 24.04 · Windows 10 · Kali Linux · Wazuh · Sysmon · Atomic Red Team · MITRE ATT&CK · Python · Anthropic API

## About

Built by Ethan, a high school student in Texas, summer 2026. Project for skill-building going into senior year.