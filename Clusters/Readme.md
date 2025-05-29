# VMware Cluster with vCenter Server - Standard Operating Procedure (SOP)

## Overview
This document outlines the step-by-step process for building a VMware cluster consisting of three ESXi hosts and a vCenter Server instance using VMware Workstation.

## Purpose
The SOP provides a standard deployment of a VMware cluster for Electric Petrol's first data center implementation. It provides clear instructions for technicians to follow.

## Key Components
- **ESXi Hosts**: Three hosts configured with VMware ESXi.
- **vCenter Server**: Centralized management for the VMware cluster.
- **Network Configuration**: Detailed IP allocations for management, vMotion, and VM networks.
- **Storage and High Availability**: Shared storage and optional High Availability configurations.

## Prerequisites
### Software
- VMware Workstation 16/17 Pro
- ESXi 6.7/7.0 installation ISO
- vCenter Server Appliance (VCSA) ISO

### Hardware
- Host machine with at least 32GB RAM
- 200GB of free disk space
- Intel VT-x/AMD-V enabled in BIOS

### Network
- Static IP addresses for ESXi hosts, vCenter Server, and DNS/NTP servers.
- Private network segment for management.

## Procedure
1. **Prepare the Environment**  
   - Create a custom virtual network in VMware Workstation.
   - Validate ISO checksums.

2. **Deploy ESXi Hosts**  
   - Configure three ESXi VMs with unique hostnames and IPs.

3. **Deploy vCenter Server**  
   - Install and configure vCenter Server with static IP and SSO domain.

4. **Create VMware Cluster**  
   - Set up a Datacenter and cluster in vCenter, enabling Disaster Recovery and High Availability.

5. **Configure Storage and Network**  
   - Add shared datastores and configure standard/distributed switches.

6. **Final Configuration**  
   - Set up NTP, syslog, and apply security policies.

## Acceptance Tests
- Verify all ESXi hosts are visible in vCenter.
- Test VM creation, vMotion, and High Availability.
- Ensure backup system accessibility.

## Output
- Document IPs/hostnames.
- Create build book with cluster specs and network diagram.
- Handover documentation to operations team.

## Appendices
- **Troubleshooting**: Common issues and resolutions.
- **Common Issues and Fix Scripts**: PowerCLI/ESXi shell commands for quick fixes.
- **Cross-Site Connectivity & Failover**: Procedures for DR and test environments.

## Revision History
- **Version 1.0**: Initial release (29 May 2025).

