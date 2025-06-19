# Technical Design Report: Virtualization Cluster Upgrade for electric-petrol.ie

**Author:** Philip Stewart
**Date:** June 19, 2025

## Introduction

This report outlines the technical steps and procedures required to 
upgrade the existing two-host virtualization setup at each branch site of 
electric-petrol.ie to a three-node Hyper-V failover cluster with shared storage. 

[cite_start]The primary goal of this upgrade is to ensure high availability (HA) in case of a single server failure, significantly improving the resilience of critical services. 

## Key Enhancements

The proposed solution includes the following key improvements:
* [cite_start]**Expansion to a Three-Node Cluster:** Adding a third host server for a more robust and redundant cluster. 
* [cite_start]**Implementation of Clustered Shared Storage:** Utilizing Storage Spaces Direct or an alternative shared storage solution to facilitate seamless failover. 
* [cite_start]**Enhanced High Availability (HA):** Guaranteeing continuous operation for critical Virtual Machines (VMs) even if one host fails. 
* [cite_start]**Improved AAA Integration:** Re-engineering Authentication, Authorization, and Accounting (AAA) services for tighter integration with the electric-petrol.ie domain. 
* [cite_start]**Additional Technical Benefits at No Extra Cost:** Leveraging existing Windows Server Datacentre licensing for features like live migration, improved redundancy, and streamlined management. 

## Current Infrastructure Overview

The current virtualization infrastructure at each branch consists of two Dell R440 servers with specific configurations for CPU, RAM, OS and data storage, networking, and Hypervisor (Windows Server 2019 Datacentre). [cite_start]Each host runs various Virtual Machines (VMs) including Domain Controllers, File Servers, SQL Servers, and Syslog servers. 

## Customer Requirements

The upgrade addresses the following key requirements from electric-petrol.ie:
* [cite_start]Adding a third host server for enhanced fault tolerance. 
* [cite_start]Implementing Clustered Shared Storage to enable Hyper-V HA features. 
* [cite_start]Ensuring continuous operation of critical services if one server fails. 
* [cite_start]Re-engineering AAA services for modern integration with the electric-petrol.ie domain. 

## Proposed Upgrade Plan

[cite_start]This section details the comprehensive plan, covering new server specifications, storage solutions, network adjustments, Hyper-V cluster configuration, and AAA integration. 

### A. New Server Specifications (Dell R450 Recommended)
[cite_start]The new third server will largely match the existing infrastructure (Dell PowerEdge R450s), with minor strategic upgrades for future growth and performance, including larger OS drives and increased data storage capacity or performance. 

### B. Storage Upgrade (Clustered Shared Storage)
Clustered shared storage is fundamental for Hyper-V High Availability. Two options are recommended:
* [cite_start]**Option 1: Storage Spaces Direct (S2D):** Requires at least two SSDs per node for caching, offering resilient, distributed storage with excellent performance and scalability. 
* [cite_start]**Option 2: External SAN/iSCSI Storage:** A more traditional approach, potentially using a Dell MD3420, connecting via iSCSI. 

### C. Network Adjustments for High Availability
Optimized network configuration is crucial:
* [cite_start]**Dedicated 10Gb/s Network:** Provisioned for cluster heartbeat and storage traffic to minimize latency and maximize throughput. 
* [cite_start]**VLAN Segmentation:** Implemented for management, storage, and VM traffic to enhance security, isolate traffic, and improve performance. 

### D. Hyper-V Cluster Configuration
The Hyper-V cluster will be configured for maximum high availability:
* [cite_start]Failover Clustering will be enabled on all three nodes. 
* [cite_start]Cluster Shared Volumes (CSV) will be set up for concurrent VM storage access. 
* [cite_start]VM priorities will be configured (e.g., DCx and DBx as high availability) for critical VM restarts. 
* [cite_start]Live Migration will be enabled for seamless VM transfers between nodes. 

### E. AAA Integration with electric-petrol.ie
AAA services will be re-engineered for robust integration:
* [cite_start]Migration to Active Directory-integrated RADIUS (NPS) for centralized management. 
* [cite_start]Time synchronization across all servers via the existing Galleon NTS-6002-GPS to prevent authentication issues. 

## High Availability & Failover Strategy

The core of this upgrade establishes a robust high availability and failover strategy:
* [cite_start]**Automatic VM Restart:** VMs will automatically restart on surviving nodes after a host failure. 
* [cite_start]**Node Majority Configuration:** The three-node cluster will be configured with a Node Majority setup, meaning that at least two nodes must be online and communicating for the cluster to operate, thereby providing resilience against individual node failures. 
* [cite_start]**Priority Settings:** Critical VMs (DCx, DBx) will have high priority for faster restarts during failover. 

## Additional Technical Benefits at No Extra Cost

Leveraging existing Windows Server 2019 Datacentre licensing, the upgrade provides:
* [cite_start]**Live Migration:** Seamless VM movement without downtime. 
* [cite_start]**Storage QoS (Quality of Service):** Prevents individual VMs from monopolizing storage resources. 
* [cite_start]**Better Monitoring:** Centralized management via Windows Admin Centre. 
* [cite_start]**Backup Improvements:** Enhanced backup capabilities via Azure Backup Server. 

## Implementation Steps

The upgrade will be carried out in a structured manner, following these key implementation steps:
1.  [cite_start]Procure & Deploy Third Server. 
2.  [cite_start]Upgrade Existing Hosts (SSD cache, firmware/driver updates). 
3.  [cite_start]Configure Networking (dedicated 10Gb/s network). 
4.  [cite_start]Set Up Failover Cluster & Shared Storage. 
5.  [cite_start]Migrate VMs to Cluster Shared Volumes. 
6.  [cite_start]Test Failover Scenarios. 
7.  [cite_start]Integrate AAA into Active Directory. 

## Risks & Mitigations

Potential risks have been identified with corresponding mitigation strategies:
* [cite_start]**Storage performance bottleneck:** Implement SSD caching (with S2D) or select a performant SAN solution. 
* [cite_start]**Network congestion:** Utilise a dedicated 10Gb/s storage network and VLAN segmentation. 
* [cite_start]**VM boot storm during failover:** Implement staggered VM startup priorities. 

## Implementation Strategy Across Multiple Sites

[cite_start]A phased approach tailored to each site's specific requirements will be employed to minimize disruption. 

### Sites 1 and 2: Full Replacement with Parallel Build
* [cite_start]A complete refresh of server hardware is recommended, acquiring three identical new Dell PowerEdge R450s for each site. 
* [cite_start]A new three-node Hyper-V cluster will be built in parallel to the existing two-host setup for thorough testing without impacting live operations. 
* [cite_start]A carefully planned cutover of VMs to the new cluster will occur once verified, with a failback plan as contingency. 
* [cite_start]Existing Dell R440s will be recovered for other uses or decommissioning after a successful cutover. 

### Site 3: Phased Integration with Existing Spares
* [cite_start]Existing spare servers (four available) will be leveraged to form the new three-node cluster, offering a cost-effective solution. 
* [cite_start]The new cluster will be built in parallel to the current setup, ensuring no downtime. 
* [cite_start]A planned cutover will move VMs to the new cluster, with a comprehensive failback strategy. 
* [cite_start]Existing Dell R440s will be recovered for redeployment or other purposes upon successful migration. 

### Site 4: Optimized Integration with Fewer Spares
* [cite_start]Three spare servers will be directly configured to form the new three-node Hyper-V cluster. 
* [cite_start]The new cluster will be built in parallel to the existing environment, ensuring minimal disruption. 
* [cite_start]A controlled cutover process will migrate VMs, backed by a robust failback plan. 
* [cite_start]Existing Dell R440s will be recovered once the new cluster is fully operational and stable. 

## Conclusion

[cite_start]This upgrade represents a significant enhancement to the reliability, performance, and manageability of critical IT services for electric-petrol.ie.  Upon completion, the new infrastructure will provide:
* [cite_start]**High Availability:** Ability to survive single node failures. 
* [cite_start]**Better Performance:** Optimized shared storage and dedicated 10Gb networking. 
* [cite_start]**No Extra Licensing Costs:** Leveraging existing Windows Server Datacentre benefits. 
* [cite_start]**Simplified Management:** Streamlined administration with Windows Admin Centre. 

## Next Steps

[cite_start]The report recommends approving the server purchase and scheduling a suitable downtime window for the migration process from the old infrastructure to the new.