# Network Design

This document describes the planned network topology and per-VM specifications for the home SOC lab. The design is locked here before any VMs are built — the goal is to think through the architecture once, document it, and then implement against the doc.

## Hypervisor Choice

**Hypervisor:** UTM (QEMU backend) on macOS

**Why UTM and not VirtualBox?** The host machine is an Apple Silicon Mac (M-series). VirtualBox's support for non-ARM guest operating systems on Apple Silicon is unreliable — x86_64 ISOs frequently fail to boot in the UEFI firmware stage. UTM provides native ARM64 virtualization via Apple's Hypervisor framework, which yields near-native performance for ARM-architecture guests and stable booting. All guest operating systems in this lab are therefore ARM64 builds.

This is a deliberate engineering tradeoff: native virtualization on Apple Silicon requires ARM64 guests, which is well-supported by Ubuntu, Kali, and Windows 11 ARM.

## Topology

A single isolated /24 network running on UTM in Host-Only mode. The Mac host acts as the only bridge between the lab and the outside world, and only when explicitly needed for software installation.
+--------------------------------+
              |      Mac Host (32 GB RAM)      |
              |  UTM + Shared Host-Only Network|
              +---------------+----------------+
                              |  192.168.64.1
                              |
             +----------------+----------------+
             |                |                |
    +--------+-----+ +--------+-----+ +--------+-----+
    | wazuh-server | | win11-victim | | kali-attacker|
    | Ubuntu 24.04 | |  Windows 11  | |  Kali Linux  |
    |   ARM64      | |     ARM64    | |    ARM64     |
    | 192.168.64.10| | 192.168.64.20| | 192.168.64.30|
    |   SIEM/IDS   | |   Endpoint   | |   Attacker   |
    +--------------+ +--------------+ +--------------+
                   (Host-Only network 192.168.64.0/24)
                   ## Network Configuration

| Item | Value |
|------|-------|
| Network CIDR | 192.168.64.0/24 |
| Network Mode | UTM Host-Only |
| Subnet Mask | 255.255.255.0 |
| Host Adapter IP | 192.168.64.1 |
| DHCP | Used for initial install; static IPs assigned post-install |
| Internet Access | None by default (intentional isolation) |

## Virtual Machine Specifications

| VM | OS | Hostname | IP | RAM | Disk | vCPU | Role |
|----|----|----------|-----|-----|------|------|------|
| 1 | Ubuntu Server 24.04 LTS (ARM64) | wazuh-server | 192.168.64.10 | 4 GB | 50 GB | 2 | SIEM (Wazuh manager + indexer + dashboard) |
| 2 | Windows 11 Enterprise ARM (90-day eval) | win11-victim | 192.168.64.20 | 4 GB | 60 GB | 2 | Monitored endpoint (Sysmon + Wazuh agent) |
| 3 | Kali Linux ARM64 | kali-attacker | 192.168.64.30 | 4 GB | 30 GB | 2 | Offensive tooling for simulated attacks |

**Total resource commitment:** 12 GB RAM, 8 vCPU, 140 GB disk.
**Host capacity:** 32 GB RAM, well within budget.

## Why Host-Only Networking?

A host-only network creates an isolated virtual subnet that exists only between the Mac host and the VMs. Three reasons this matters:

1. **Containment.** When Kali simulates attacks against win11-victim, no traffic ever touches the home Wi-Fi. There is no risk of accidentally scanning a neighbor's device or tripping the home router's intrusion detection.
2. **Reproducibility.** Static IPs and a known subnet mean every detection rule, threat-hunt query, and screenshot in this project is reproducible.
3. **Realistic threat model.** Real corporate SOC environments are heavily segmented. Host-only mirrors that posture: assume hostile actors live on the network and design accordingly.

When internet access is needed (downloading Wazuh updates, Sysmon installer, Atomic Red Team modules), the VM's network mode is temporarily switched to Shared (UTM's NAT equivalent), the download is performed, and the mode is switched back to Host-Only.

## IP Allocation Plan

Reserved address blocks within 192.168.64.0/24:

| Range | Purpose |
|-------|---------|
| .1 | Mac host (UTM-assigned) |
| .10–.19 | Defensive infrastructure (SIEM, future log forwarder, future SOAR) |
| .20–.29 | Monitored endpoints |
| .30–.39 | Offensive / attacker systems |
| .100–.200 | Available for future expansion (e.g., honeypots, additional victims) |

## ARM64 vs x86 — Notes for the SOC Lab

All standard SOC tooling used in this lab supports ARM64:

- **Wazuh:** official ARM64 packages for both server and agent
- **Sysmon for Windows ARM:** Microsoft ships Sysmon for ARM Windows 11
- **Atomic Red Team:** PowerShell-based, architecture-agnostic
- **Anthropic API client (Python):** architecture-agnostic

No detection rules, MITRE mappings, or threat-hunt techniques change based on guest architecture. The lab's analytical work is identical on ARM64 and x86_64.

## Out of Scope (for now)

The following are intentionally not part of the initial design but are candidates for future expansion:

- pfSense or OPNsense firewall VM as an inline gateway
- Active Directory domain controller for more realistic enterprise simulation
- Suricata or Zeek for network-based detection
- A second log source (Linux victim) to broaden detection coverage