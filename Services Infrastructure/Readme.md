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

####################################################################################################################################################################################################################################
# ServicesDescription.md
**Written By:** Philip Stewart 
**Student Number:** L00187850
**Project:** Hybrid Cloud Technology Assignment 2
**Date:** 19 May 2025 
**Audience:** technical audience


This document outlines the infrastructure services deployed across the Hyper-V servers to support the Electric Petrol project.

## Active Directory (AD DS)

- **Domain Services**: One Domain Controller per server for high availability (Primary and Secondary)
- **VLAN Association**: VLAN 2 (Hosts VLAN)
- **DNS Integration**: Internal DNS zones hosted via AD-integrated DNS

### Steps to set up DNS
1. Open Server Manager
2. Add Roles & Features - In Server manager go to **Manage** and then **Add roles & Features**
3. Select Server Roles - Choose active directory domain services and click **Add Features**.
4. Review and Install - Review the installation reqirements which come up and then click **Install**

## Domain Controller
It's usually best practice to have the Primary and Secondary Domain controllers virtualized. This allows for increased efficiency, improved security and enhanced flexibility.

A VLAN has been created specifically for domain controllers to increase security as the DCs would require an external adapter to function correctly.

Internet access would be required to allow both internal and external name resolution for DNS. This would be removed if the site didn't resolve IPs from external sources.
### Steps to promote a server to a domain controller can be found below:

1. **Post Installation Notification** - Once AD DS is install then a notification flag appears in Server Manager. click on it and select **Promote this server to a domain controller.**
2. **Choosing DC options** - Select **Add a new forest** to create a new domain. 
3. **Configure Domain Details** - Enter the root domain name (for this exercise it would be **corp.electric-petrol.ie**) and the Directory Services Restore Mode Password.
4. **DNS Configuration** - Accept the default NetBIOS domain name and accept the defaults provided.
5. **Reboot** - After reboot, the domain will function correctly.

## Domain Name System (DNS)

- **DNS Zones**:
  - Internal domain: `corp.electric-petrol.ie`
  - Forwarders configured to external DNS (e.g., Google 8.8.8.8 or cloudflare at 1.1.1.1)
- **Redundancy**:
  - Dual DNS role holders on separate Hyper-V hosts - One running on the primary DC and one on the secondary DC.
- **VLAN Association**: VLAN 2 and VLAN 6 (Infrastructure Servers VLAN)

### Steps to set up DNS
1. Open Server Manager and click **Manage < Access Roles and Features.**
2. Select **Role-based or Feature-based installation.**
3. Select the local server.
4. Select DNS Server.
5. Click Next and Installed.

		**Configuration**
1. Open DNS Manager (dnsmgmt.msc)
2. Expand the server in the console and right click **Forward Lookup Zones < New Zone.**
3. Select **Primary Zone**, **Store in Active Directory** (only if DNS is on the domain controller), Replication Scope to **all DNS Servers in this forest or domain**, **The Zone name is corp.electric-petrol.ie**, And **Allow Only Secure Dynamic Updates.**

		**Forwarder Set up**
1. In DNS Manager right click the server and select **Properties**
2. Click the **Forwarders** and click **Edit**
3. Add IP addresses such as public resolvers (1.1.1.1 or 8.8.8.8) and internal services such as the firewall or application server. Click Ok.

		**set as default DNS server for end devices - Manual Method**
1. Open Network Adapter < IPV4 Settings < Preferred DNS=IP of DC/DNS server.
	a) can also be configured on the router side to allow for automatic device discovery.

## Dynamic Host Configuration Protocol (DHCP)

- **Scope Design**:
  - Scope for internal clients
  - Exclusions for static IP devices (i.e., servers and switches)
- **Failover Configuration**:
  - DHCP Failover (Hot Standby or Load Balanced mode)
- **VLAN Association**: VLAN 5 (Client VLAN) and VLAN 6 (Servers VLAN)

### Steps to set up DHCP
1. Open Server Manager
2. Click Manage < **Add Roles and Features**
3. Select **Role Based or Feature-Based Installation**, select the local server, under **Server Roles** click **DHCP Server**
4. Click next and install.

### Authorize the DHCP Server in AD
1. Open DHCP Console (dhcpmgmt.msc)
2. right click the server node < **Authorise**)

### Create a new DHCP Scope
1. Open DHCP Console, Expand the server < right click IPV4, **New Scope**
2. **Scope Name:** CorporateLAN, **Start IP:** 10.32.3.100, **End IP:** 10.32.3.199 (Staff devices - give multiple scopes for multiple vlans), **Subnet Mask:** (255.255.255.0), **Default Gateway:** 10.32.3.1, **DNS Server:** 10.32.1.1, **Lease duration:**  8 Days
3. right click the new scope - Activate.
4. Configure Exclusions where appropriate (i.e DNS Servers, Domain Controllers, Network Switches/routers, certain Infrastructure Servers etc)

## Network Time Protocol (NTP)

- **Internal Time Source**: DCs configured with a reliable external NTP server
- **Clients**: All devices (servers, switches, clients) synchronize with AD-integrated NTP
- **VLAN Association**: Accessible from all VLANs (routed or trunked as necessary)

### Configure NTP
1. Open Server Manager
2. Click Manage < **Add Roles and Features**
3. Select **Role Based or Feature-Based Installation**
4. Select the local Server.
5. Expand Features and check **Windows Time Service** 
6. Click Next and Install.
		**Configuring Windows Time Service**
1. Open CMD as Administrator
2. Run the command "w32tm /config /manualpeerlist:"NTP Server IP Address or Hostname" /syncfromflags:manual /reliable:YES /update" where the server IP address would be 10.32.1.1 - the Domain Controller.
3. Restart the Service using the command "net stop w32time && net start w32time"
4. To verify connection run the command "w32tm /query /status"
5. Enable a firewall rule to all traffic on UDP Port 123 for the NTP protocol.

## Management

- VLAN 10 is reserved exclusively for remote troubleshooting and management traffic.
- All service monitoring is conducted via management interfaces (VLAN 10).

## VLAN Overview
|------|----------------------------------------|
| VLAN | Purpose                      		    |
|------|----------------------------------------|
| 1    | Domain Controllers 					|
| 2    | Hosts VLAN	(for Hyper-V hosts)			|
| 3    | Staff Devices VLAN			      		|
| 4    | Power Infrastructure VLAN            	|
| 5    | Technical Clients VLAN               	|
| 6    | Infrastructure Servers VLAN  	    	|
| 7    | Hosts VLAN (for Hyper-V hosts)   		|
| 8    | HVAC Infrastructure VLAN  				|
| 9    | Storage VLAN      						|
| 10   | Network Management VLAN                |
| 11   | Security Cameras VLAN               	|
| 12   | Wireless Access Controller VLAN  		|
| 13   | Internal chat application VLAN  		|
| 14   | Loopbacks VLAN  						|
| 15   | Routers VLAN  							|
| 16   | Links VLAN  							|
| 999  | Native VLAN  							|
| 1000 | Links VLAN  							|
|------|----------------------------------------|
####################################################################################################################################################################################################################################
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
####################################################################################################################################################################################################################################
