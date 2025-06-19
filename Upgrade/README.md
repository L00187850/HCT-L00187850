# Technical Design Report: Virtualization Cluster Upgrade for electric-petrol.ie

**Author:** Philip Stewart
**Date:** June 19, 2025

|------------------------------------------------------------------------|
|								**Description**							 |
|The document within this repo (both the docx file and the PDF) details  |
|a plan to upgrade electric-petrol.ie's current two-host virtualization  |
|setup to a three-node Hyper-V failover cluster with shared storage.	 |
|																		 |
|The primary aim is to significantly enhance system resilience and ensure|
|high availability for critical services by preventing interruptions in  |
|case of a single server failure.										 |
|																		 |
|This involves expanding to a three-node cluster, implementing clustered |
|Shared Storage,optimizing network configurations, and re-engineering AAA| 
|services for better Active Directory integration.						 |
|																		 |
|The upgrade also leverages existing Windows Server Datacentre licensing |
|for additional benefits like live migration and simplified management	 |

