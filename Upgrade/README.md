# Technical Design Report: Virtualisation Cluster Upgrade for electric-petrol.ie

**Author:** Philip Stewart
**Date:** June 19, 2025

## Document Overview: 

This document, formally known as the "Technical Design Report: Virtualisation Cluster Upgrade for electric-petrol.ie", outlines a detailed plan to transform electric-petrol.ie's current two-host virtualisation setup into a more robust three-node Hyper-V failover cluster with shared storage.

The core objective is to significantly enhance system resilience and ensure continuous availability of critical services by mitigating the impact of single server failures.

The report comprehensively covers various aspects of the proposed upgrade, including:
* **New Infrastructure:** Detailed server specifications, storage solutions (like Storage Spaces Direct or SAN), network adjustments for high availability, and Hyper-V cluster configurations.
* **AAA Integration:** Re-engineering Authentication, Authorisation, and Accounting (AAA) services for better integration with Active Directory.
* **Implementation Strategy:** Step-by-step implementation procedures, identified risks and their mitigation strategies, and tailored deployment approaches for multiple branch sites.

The upgrade also leverages existing Windows Server Datacentre licensing, providing additional benefits like live migration and simplified management without extra cost.