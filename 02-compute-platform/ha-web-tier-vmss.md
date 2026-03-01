# High Availability Web Tier with Virtual Machine Scale Set

**Scope:**  
High availability and autoscaling web/application tier running on Azure Virtual Machine Scale Sets (VMSS), aligned with:

- `00-foundation/management-group-design.md`
- `00-foundation/rbac-architecture.md`
- `00-foundation/policy-baseline.md`
- `01-network-architecture/hub-spoke-topology.md`
- `02-compute-platform/data-platform-landing-zone.md` (for integration where relevant)

This document defines the architecture, availability model, scaling strategy, and operational approach for a stateless web/API tier.

---

## 1. Design Objectives

- Provide resilient web/API tier across fault and update domains
- Support horizontal autoscaling based on load
- Integrate with the hub-spoke network for secure access
- Enforce enterprise governance controls (policy, RBAC, diagnostics)
- Enable repeatable deployment via Infrastructure as Code

Assumption:  
Workload is **stateless** at the compute tier; state is stored in external services (database, cache, storage).

---

## 2. Architecture Overview

High level:

```text
Internet
   │
   │  (HTTPS only)
   ▼
Public Load Balancer / Application Gateway (WAF)
   │
   ▼
VM Scale Set (Web/API VMs)
   │
   ▼
Data Services (DB, Cache, Storage, APIs)

```

## 3. Compute and Availability Design

Virtual Machine Scale Sets (VMSS) are selected to support horizontal scaling and uniform configuration management for stateless web/API workloads.

Where supported, the VMSS is deployed across at least two Availability Zones to improve resilience against zonal failure and increase SLA posture.

In regions without zone support, Availability Sets are used to distribute instances across fault and update domains.

Production workloads use versioned images from a Shared Image Gallery to ensure consistency and enable controlled rolling upgrades.


## 4. Network Integration

The VMSS is deployed into the `app-subnet` within a spoke VNet aligned to the hub-spoke topology.

Inbound traffic flows:

Internet → WAF or Load Balancer → VMSS backend pool

Outbound traffic flows:

VMSS → User Defined Route → Azure Firewall in Hub → Internet

Private dependencies such as databases and storage accounts are accessed via Private Endpoints and resolved using centralized Private DNS.


## 5. Autoscale Strategy

Autoscale policies are defined using performance metrics such as CPU utilization and application request load.

Baseline configuration includes:

- Minimum instance count to maintain availability
- Scale-out triggered by sustained load thresholds
- Scale-in triggered by sustained low utilization
- Cooldown periods to prevent oscillation

This ensures elasticity while maintaining cost control.


## 6. Administration and Access

Administrative access follows enterprise governance controls.

There are no public IPs assigned to VMSS instances.

Access is provided through Azure Bastion or Just-In-Time access.

Role assignments are scoped to the resource group level and privileged roles are protected using PIM.


## 7. Backup and Recovery Strategy

The web tier is designed as stateless.

Application state is externalized to managed services such as databases and storage.

Recovery of compute resources is achieved through redeployment using Infrastructure as Code and versioned images.

Backup of OS disks is only required where compliance mandates it.


## 8. Patching and Upgrade Model

Application updates are delivered through image versioning and rolling VMSS upgrades.

Operating system updates are handled through controlled maintenance windows or image refresh cycles.

Rolling upgrade policies ensure instances are validated through health probes before traffic is routed.


## 9. Security Controls

Security controls include:

- No public IP addresses on instances
- Network Security Groups restricting inbound access
- Web Application Firewall or reverse proxy in front of compute
- Defender for Cloud enabled
- Diagnostic settings enforced via Azure Policy

This enforces layered defense aligned to Zero Trust principles.


## 10. Monitoring and Observability

Monitoring includes:

- Instance health and probe status
- CPU and memory metrics
- Autoscale events
- HTTP error rates

All diagnostics and activity logs are forwarded to the centralized Log Analytics workspace defined in the governance baseline.

Alerts are configured to align with availability and performance objectives.


## 11. Cost and Trade-offs

Availability Zones increase resilience but introduce additional cost.

VMSS provides infrastructure-level control compared to PaaS alternatives but requires operational ownership.

Autoscale configuration must balance performance requirements against budget constraints.


## 12. Solution Architect Alignment

This high availability compute pattern demonstrates:

- Resilient workload design
- Elastic scalability
- Secure network integration
- Governance-aligned deployment
- Operational readiness

Combined with governance, hub-spoke networking, and the data platform landing zone, this completes the core Azure infrastructure architecture narrative aligned to Solution Architect responsibilities.