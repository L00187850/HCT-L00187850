# HostConfiguration.md
**Written By:** Philip Stewart 
**Student Number:** L00187850
**Project:** Hybrid Cloud Technology Assignment 2
**Date:** 19 May 2025 
**Audience:** technical audience

This document outlines the physical and virtual configuration of the host servers used in the Electric-Petrol.ie Hyper-V infrastructure. 
It includes specifications such as hardware, host operating system details, high availability design, licensing strategy, and justifications for technology choices.

## Physical Configuration

This infrastructure is based on the Dell PowerEdge R740 server which I have chosen for this build. The business in question is a small to medium enterprise so cost is a large factor. We are using two R740 servers (with a third available for future expansion), 
each connected to redundant power rails (230VAC A and B).

**Specifications:**
- Device Model: Dell PowerEdge R740
- Warranty: 5 years Dell prosupport
- Redundant power supply: 2x 1100W PSUs (Power Rail A & B)
- Network: Each server has ports mapped to VLANs for AD, storage, DNS, DHCP, NTP, and tagged trunk traffic
- Each server supports:
  - VLAN 2 (Hosts)
  - VLAN 5, 6, 9, 10, 12
  - RS232, VGA, USB (These ports are used for offline troubleshooting)

The servers are installed in a 47U rack cabinet with correct cable management, blanking panels to allow for airflow, and are backed by a 4kVA UPS system and Static Transfer Switch for resilience.

## Host OS
- **Operating System**: Windows Server 2022 Datacenter (licensed).
- **Hypervisor**: Microsoft Hyper-V. Used instead of VMWare to accommodate cost.
- **License Justification**:
  - Windows Server 2022 Datacenter was selected to allow for unlimited VM creation and there is no additional cost for virtualization. It includes Hyper-V and unlimited guest OS virtualization rights.
  - Ideal operating system for a virtualized infrastructure with Active Directory, DNS, DHCP, and future VM growth/expansion.

## High Availability Design

To ensure system resilience and continuous service availability, 
this infrastructure uses several layers of high availability (HA) strategies. 
These address both hardware-level redundancy and virtualization fault tolerance.

Two physical Dell PowerEdge R740 servers are configured as **Hyper-V hosts**. 
These are identical in hardware specification and act as peers in a **failover cluster**. 
This design supports the following features:

### 1. Dual Hyper-V Hosts (Dell PowerEdge R740)
- **Live Migration** of virtual machines between hosts with zero downtime using Microsofts hyper-v failover clustering.
- **Automatic failover** in the event of host failure
- **Load balancing** across hosts for performance optimization

The third server in the rack (initially unused) is reserved for future expansion or failover capacity should one of the servers fail.

A single gold image for Servers and another gold image for a domain controller would be provisioned to allow for quick configuration.
---

### 2. Storage Redundancy

Each Hyper-V host uses **local storage for performance-critical VMs** and can optionally connect to a shared storage backend (e.g., NAS/SAN) for clustered services. 

Two virtual file servers would be provisioned, one replicating to the other to allow for failover.
---

### 3. Power Redundancy
The Power infrastructure includes:

- **dual power rails** for each server and networking components:
- **Power Rail A** and **Power Rail B** connecting to two separate **4kVA UPS units**
- Power input is further protected by a **Static Transfer Switch (STS)** to prevent single points of failure

This guarantees uninterrupted power even if one UPS or power path fails.

---

### 4. Network Redundancy

The network design includes:
- **Multiple VLANs** mapped to multiple physical network interfaces
- At least **4 Ethernet interfaces per host** for separating management, host traffic, storage, and service networks
- **Management switches** and **top-of-rack switches** provide multiple paths for data and control traffic

Tagged VLAN traffic over **SFP28+ ports** enables high-speed fibre links and allows for future scalability.

---

### 5. Host OS Resilience

Each host runs **Windows Server 2022 Datacenter Edition**, which supports:
- **Failover Clustering**
- **Cluster-Aware Updating** (automatic patching with rolling reboots)
- **Health monitoring and alerting** using Windows Admin Center and SCCM (Microsoft System Centre Configuration Manager)
- **Patch Management** by using SCCM to deploy updates, applications and deployment images.

These features ensure host availability while reducing administrative downtime.

---

### 6. Dedicated Remote Management Interface and local management

- Each server includes **RS232 and VGA ports** for local troubleshooting should remote access become inacessible.
- Optional integration with **iDRAC (Integrated Dell Remote Access Controller)** provides remote access for BIOS-level diagnostics and recovery

---

### 7. KVM Access

A **KVM Monitor & Keyboard Unit** is installed in the rack for direct access to servers in the event of network or remote access failure.

---
## Operations

- Servers are monitored using Windows Admin Center and Failover Cluster Manager
- Backups and snapshots are managed via Windows Server Backup and third-party tools
- Patch management is implemented through Windows Server Update Services (WSUS) or through SCCM
