# BCDR.md
**Written By:** Philip Stewart 
**Student Number:** L00187850
**Project:** Hybrid Cloud Technology Assignment 2
**Date:** 19 May 2025 
**Audience:** technical audience


This document outlines the Disaster Recovery and Business continuity Procedures as well as a laid-out backup strategy for the Electric Petrol sites.

## Business Continuity, Backup, and Disaster Recovery Plan (BCDR)

### Backup Strategy

- **Hyper-V VMs**:
  - Full VM backups taken weekly using Windows Server Backup - old backups deleted after 180 days.
  - Stored to a secondary offsite location (when storage becomes available)
- **Domain Controllers**:
  - Active Directory-integrated backups using `wbadmin` commands and VSS (Volume Shadow Copy Service)
- **Configuration**:
  - Weekly system state backups on at least one domain controller.
  - Daily incremental data backups
- **Backup Target**:
  - Direct-attached or networked storage (to be added in storage expansion phase)
  - offsite- backup using an encrypted usb or through network replication.
  - If desired cloud backup with Azure backup or AWS storage Gateway for independant solution.
  
### Disaster Recovery (DR)

- **Scenario: Host Failure**:
  - Hyper-V Failover Cluster guarantees HA (High Availability) for all virtual workloads.
  - Live Migration enabled with shared storage or Cluster Shared Volumes (CSV). 
  - Redundant power and dual network paths mitigate most hardware failure scenarios.
  - System logs and alerting configured to trigger on failover events
  
  
- **Scenario: Full Rack Failure**:
  - Offsite VM exports and configuration backups enable recovery to alternate hardware
  - Use Azure or another cloud provider for DR (future phase)
  - Restore in this order: Domain Controller < DNS < DHCP < Critical Application Servers
  - Disaster Recovery rehearsal conducted using Hyper-V’s Import VM and offline testing. I.e ran in a lab/test setting.
 
- **DNS and NTP in Recovery:
  - NTP Settings Reapplied using Registry Scripts if the domain controller roles are rebuilt.
  - Internal DNS guarantees that clients and services which are restored can register and resolve internally rather than externally.


### Business Continuity (BC)

- **Redundant UPS (4kVA each) and Static Transfer Switch**:
  - Immediate switch between power sources in case of failure using the Static Transfer Switch.
  - Estimated 30+ minutes battery runtime, enough for graceful shutdown
  - Critical hardware on separate power circuits where possible
  
- **Staff Procedures**:
  - All critical infrastructure changes logged
  - Clear runbooks for failover and recovery steps
- **Testing**:
  - Backups and recovery procedures tested quarterly
  - Yearly full Disaster Recovery simulation, which also includes offsite recovery
  

### Justification

- High Availability using Hyper-V Clustering allows for hassle-free failover.
- Provides protection against hardware and rack-level failures such as a loss of power using system backups and multiple power circuits.
- Time, DNS and DHCP resiliance using a multi-tiered configuration and backup schedule.

### Future Enhancements
Possible ways in the future where it could be improved would be some of the following options:

- Cloud-Based Replication for high-priority virtual machines such as DNS, DHCP etc.
- DHCP Failover across a second domain site instead of only having one.
- Automatic time sync fallback.
- Improved Integration with SIEM (System Information and Event Management), a type of software solution that helps respond to security issues; and other monitoring tools.