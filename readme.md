# Azure Infrastructure Architecture & Implementation Portfolio

This repository showcases Azure infrastructure architecture and implementation work aligned to:

- Azure Solution Architect responsibilities at Microsoft and enterprise customers
- Azure Landing Zone and Cloud Adoption Framework guidance
- Governance-first cloud design (management groups, policy, RBAC, cost)
- Hands-on implementation capabilities built from AZ-104 and beyond

## Structure

- `00-foundation` – Enterprise governance:
  - Management group hierarchy
  - RBAC architecture and role model
  - Policy and compliance baseline

- `01-network-architecture` – Hub-spoke, hybrid connectivity, secure egress and network security patterns.

- `02-compute-platform` – VM/VMSS topologies, availability sets/zones, scaling and platform services.

- `03-storage-backup-dr` – Storage tiering, backup, restore and disaster recovery design.

- `04-monitoring-operations` – Log Analytics, diagnostics, alerting, workbooks and operational runbooks.

- `Instructions` – Lab-style exercises and reference implementations used to validate designs.

## Purpose

The goal of this repository is to present a **portfolio of architecture decisions**, not just lab steps:

> Governance & architecture in `00-*` & `01-*` folders,  
> Backed by working implementations in `Instructions`.

This aligns with how a Solution Architect at Microsoft or a partner would document and defend their designs with real evidence.


---

## Architecture Overview

This portfolio represents a layered enterprise Azure architecture built using a governance-first approach.

The design follows a structured progression:

1. **Control Plane (Governance Foundation)**  
   - Management group hierarchy  
   - Role-based access control (RBAC) model  
   - Azure Policy baseline  

2. **Network Plane (Secure Connectivity)**  
   - Hub–spoke topology  
   - Centralized firewall and secure egress  
   - Private endpoint and DNS integration  

3. **Workload Plane (Compute & Data)**  
   - High availability VMSS web tier  
   - Enterprise data platform landing zone  
   - Environment isolation (Prod / NonProd)  

4. **Resiliency Layer (Storage, Backup & DR)**  
   - Tiered storage strategy  
   - Backup enforcement via policy  
   - Region-paired disaster recovery design  

5. **Operational Excellence Layer (Monitoring & Observability)**  
   - Central Log Analytics architecture  
   - Alerting model (platform, workload, application)  
   - Runbooks and incident response integration  

Each layer builds on the previous one, reflecting how enterprise Azure environments are designed, governed and operated in real-world scenarios.

---

## Architectural Flow

The repository can be read in this order:

`00-foundation` → `01-network-architecture` → `02-compute-platform` → `03-storage-backup-dr` → `04-monitoring-operations`

This mirrors how a Solution Architect would design:

Govern → Secure → Deploy → Protect → Operate

---

## Design Principles

Across all layers, the architecture consistently applies:

- Governance before workload deployment  
- Least privilege access control  
- Zero Trust network posture  
- Infrastructure as Code readiness  
- Environment isolation (Prod vs NonProd)  
- Cost-awareness and scalability trade-offs  

---

## What This Demonstrates

This repository demonstrates the ability to:

- Design enterprise Azure environments from first principles  
- Align architecture decisions to governance and compliance requirements  
- Balance availability, cost and operational complexity  
- Integrate security, networking, compute and data into a cohesive model  
- Document architectural trade-offs clearly and defensibly  

The result is not a collection of labs, but a structured portfolio of architectural decisions aligned to Solution Architect responsibilities.