# Network Design

This document describes the planned network topology and per-VM specifications for the home SOC lab. The design is locked here before any VMs are built — the goal is to think through the architecture once, document it, and then implement against the doc.

## Topology

A single isolated `/24` network running on VirtualBox in Host-Only mode. The Mac host (running VirtualBox) acts as the only bridge between the lab and the outside world, and only when explicitly needed for software installation.

```
                  ┌────────────────────────────────┐
                  │      Mac Host (32 GB RAM)      │
                  │   VirtualBox Manager + vboxnet0│
                  └──────────────┬─────────────────┘
                                 │  192.168.56.1
                                 │
                ┌────────────────┼────────────────┐
                │                │                │
        ┌───────▼──────┐ ┌───────▼──────┐ ┌───────▼──────┐
        │ wazuh-server │ │ win11-victim │ │ kali-attacker│
        │ Ubuntu 24.04 │ │  Windows 11  │ │  Kali Linux  │
        │ 192.168.56.10│ │192.168.56.20 │ │192.168.56.30 │
        │   SIEM/IDS   │ │   Endpoint   │ │   Attacker   │
        └──────────────┘ └──────────────┘ └──────────────┘
                          (Host-Only network 192.168.56.0/24)
```

## Network Configuration

| Item | Value |
|------|-------|
| Network CIDR | 192.168.56.0/24 |
| Network Mode | VirtualBox Host-Only (`vboxnet0`) |
| Subnet Mask | 255.255.255.0 |
| Host Adapter IP | 192.168.56.1 |
| DHCP | Disabled — static IPs assigned per VM |
| Internet Access | None by default (intentional isolation) |

## Virtual Machine Specifications

| VM | OS | Hostname | IP | RAM | Disk | vCPU | Role |
|----|----|----------|-----|-----|------|------|------|
| 1 | Ubuntu Server 24.04 LTS | `wazuh-server` | 192.168.56.10 | 4 GB | 50 GB | 2 | SIEM (Wazuh manager + indexer + dashboard) |
| 2 | Windows 11 Enterprise (90-day eval) | `win11-victim` | 192.168.56.20 | 4 GB | 60 GB | 2 | Monitored endpoint (Sysmon + Wazuh agent) |
| 3 | Kali Linux | `kali-attacker` | 192.168.56.30 | 4 GB | 30 GB | 2 | Offensive tooling for simulated attacks |

**Total resource commitment:** 12 GB RAM, 8 vCPU, 140 GB disk.
**Host capacity:** 32 GB RAM, well within budget.

## Why Host-Only Networking?

A host-only network creates an isolated virtual subnet that exists only between the Mac host and the VMs. Three reasons this matters:

1. **Containment.** When Kali simulates attacks against `win11-victim`, no traffic ever touches the home Wi-Fi. There's no risk of accidentally scanning a neighbor's device or tripping the home router's intrusion detection.
2. **Reproducibility.** Static IPs and a known subnet mean every detection rule, threat-hunt query, and screenshot in this project is reproducible. No DHCP leases shifting addresses overnight.
3. **Realistic threat-model.** Real corporate SOC environments are heavily segmented. Host-only mirrors that posture: assume hostile actors live on the network and design accordingly.

When internet access is needed (downloading Wazuh updates, Sysmon installer, Atomic Red Team modules), the VM's network adapter is temporarily switched to NAT, the download is performed, and the adapter is switched back to Host-Only.

## IP Allocation Plan

Reserved address blocks within `192.168.56.0/24`:

| Range | Purpose |
|-------|---------|
| .1 | Mac host (VirtualBox-assigned) |
| .10–.19 | Defensive infrastructure (SIEM, future log forwarder, future SOAR) |
| .20–.29 | Monitored endpoints |
| .30–.39 | Offensive / attacker systems |
| .100–.200 | Available for future expansion (e.g., honeypots, additional victims) |

## Out of Scope (for now)

The following are intentionally not part of the initial design but are candidates for future expansion:

- pfSense or OPNsense firewall VM as an inline gateway
- Active Directory domain controller for more realistic enterprise simulation
- Suricata or Zeek for network-based detection
- A second log source (Linux victim) to broaden detection coverage