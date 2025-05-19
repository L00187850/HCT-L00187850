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