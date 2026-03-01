# Storage, Backup and Disaster Recovery Architecture

**Scope:**  
Enterprise storage, backup and disaster recovery (DR) architecture aligned with:

- `00-foundation/management-group-design.md`
- `00-foundation/rbac-architecture.md`
- `00-foundation/policy-baseline.md`
- `01-network-architecture/hub-spoke-topology.md`
- `02-compute-platform/data-platform-landing-zone.md`
- `02-compute-platform/ha-web-tier-vmss.md`

This document defines how storage is structured, protected and recovered across the environment, including patterns for backup and disaster recovery.

---

## 1. Design Objectives

- Provide secure, tiered storage aligned to workload needs
- Protect critical data with clearly defined RPO/RTO targets
- Support regional failure scenarios using Azure region pairs
- Integrate backup and DR with governance, policy and RBAC
- Minimize recovery complexity through standardized patterns

Assumptions:

- Production workloads require higher resilience and retention than non-production.
- Business impact assessments define RPO / RTO targets per application.

---

## 2. Storage Architecture

Storage is organized by workload and environment, following the management group and subscription structure defined in the foundation documents.

Key principles:

- Data is stored in the region of primary workload execution.
- Storage accounts are isolated per workload or domain (no “mega” shared accounts).
- Naming and tagging conventions follow enterprise standards for ownership and cost allocation.
- Public network access is disabled; access flows through Private Endpoints and hub-spoke networking.

Storage tiers:

- Hot: frequently accessed, low-latency data
- Cool: infrequently accessed but available data
- Archive: long-term retention at lowest cost (where retrieval latency is acceptable)

Data classification (e.g., public, internal, confidential) is mapped to encryption, access and retention requirements.

---

## 3. Backup Strategy

Backups are defined at service level, but coordinated through a central policy approach.

### 3.1 Virtual Machines and VM Scale Sets

- Protected by Azure Backup using Recovery Services vaults.
- Backup policies differ for prod and non-prod:
  - Non-prod: daily backups with shorter retention.
  - Prod: daily or more frequent backups with longer retention (e.g., 30–90 days) and weekly/monthly retention as required.
- Application-consistent backups are used where supported.

### 3.2 Databases and Data Stores

- Managed databases (SQL, Cosmos DB, etc.) use built-in backup capabilities with configured retention aligned to data classification.
- For PaaS services without native backup, patterns such as export jobs, snapshots or geo-redundant configurations are used.
- Data lake and blob storage use:
  - Soft delete for blobs and containers
  - Versioning where appropriate
  - Immutable storage (WORM) for regulatory or audit data

### 3.3 Central Governance

- Backup policies are standardized and documented.
- Recovery Services vaults are regionally aligned and centrally visible.
- Backup enablement is enforced through Azure Policy where possible (e.g., “VMs must be associated with a backup policy”).

---

## 4. Disaster Recovery Strategy

DR is based on Azure region pairs and workload criticality.

### 4.1 Region Pairing

- Each production workload is assigned a primary and paired region.
- Services that support geo-redundant capabilities (e.g., GRS storage, geo-replication for databases) are configured accordingly.
- Cross-region replication is designed to meet RPO/RTO targets; not all workloads require active-active DR.

### 4.2 DR Patterns

Common patterns:

- Cold standby:
  - Infrastructure is defined as code.
  - Only storage and critical state are replicated.
  - Compute is deployed on demand during DR.
- Warm standby:
  - Minimal capacity is running in secondary region.
  - Scales out during DR.
- Active-active (rare, high-cost):
  - Workloads run in both regions simultaneously.
  - Requires careful data consistency design.

The chosen pattern per workload is documented with business owners.

### 4.3 Failover and Recovery

- DR runbooks define the steps to fail over to secondary region and fail back.
- DNS and routing changes are planned in advance (e.g., Traffic Manager or Front Door where used).
- DR tests are conducted periodically to validate assumptions and update documentation.

---

## 5. Policy and Governance Integration

Policy baseline enforces:

- Storage account encryption at rest using Microsoft-managed keys by default; customer-managed keys (CMK) where required.
- Soft delete and minimum retention periods for blobs and containers.
- Diagnostic settings sending storage logs to central Log Analytics.
- Restrictions on public network access and unencrypted protocols.

Backup and DR posture is reviewed as part of governance reviews alongside RBAC and security posture.

---

## 6. Security and Access Controls

Security measures include:

- Private Endpoints for storage access from VNets.
- RBAC roles scoped to resource group or storage account level (e.g., Storage Blob Data Contributor, Storage Blob Data Reader).
- Key Vault for secrets and keys; access via Managed Identities where possible.
- Defender for Storage enabled for production accounts.

Access principles:

- No shared keys used in application code; SAS and Managed Identities preferred.
- Least-privilege roles assigned to service principals and users.
- Break-glass procedures for emergency access are documented and PIM-controlled.

---

## 7. Monitoring, Testing and Validation

Monitoring includes:

- Backup success/failure alerts from Recovery Services vaults.
- Storage capacity and transaction metrics.
- Alerts for anomalous activity (via Defender for Storage).
- Log Analytics queries to validate backup coverage (e.g., VMs not protected).

Testing and validation:

- Periodic restore tests from backup to a non-production environment.
- DR tests simulating regional failure where feasible.
- Review of RPO/RTO compliance against actual restore times.

---

## 8. Trade-offs and Solution Architect Alignment

Design trade-offs considered:

- Geo-redundant vs locally-redundant storage (cost vs resilience).
- Backup frequency and retention (RPO/RTO vs storage cost).
- Cold vs warm vs active-active DR patterns (complexity and cost vs recovery time).

This storage, backup and DR architecture demonstrates:

- Alignment with enterprise governance and security controls.
- Clear separation of concerns between primary operations, backup and DR.
- Practical patterns for protecting both infrastructure and data workloads.

Together with the governance foundation, hub-spoke networking, data platform landing zone and high availability compute patterns, it completes the core infrastructure resiliency story expected at Solution Architect level.