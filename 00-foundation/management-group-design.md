# Azure Management Group Design  
**Scope:** Azure Infrastructure Governance (Data Platform Foundation & Core Infrastructure)

## 1. Context and Objectives

This document defines the management group (MG) hierarchy and governance model for the Azure environment of **Contoso Health**. The design provides:

- A scalable structure for **multiple subscriptions** and environments (dev/test/prod).
- Centralized governance using **Azure Policy**, **RBAC**, and **cost management**.
- A foundation for **data platform landing zones** and other workload landing zones.

The design follows Microsoft Cloud Adoption Framework (CAF) guidance for Azure landing zones and management groups.

### Key goals

- Separate **platform** and **workload** responsibilities.
- Enforce **security, compliance, and logging** at parent scopes.
- Support future **multi-region** and **data sovereignty** needs.
- Keep the structure simple enough for small teams, but scalable for growth.

---

## 2. Design Principles

1. **Root is management, not deployment**
   - Tenant root (`Tenant Root Group`) is used for a few global policies only (e.g., allowed locations).
   - Day-to-day operations happen below an **intermediate root** management group.

2. **Environment separation**
   - Production and non-production workloads are separated at the **management group level**, not just by tags or naming.

3. **Platform vs Landing Zones**
   - **Platform** subscriptions (identity, management, connectivity) are isolated from **application/data landing zones**.
   - Workloads consume shared platform services (DNS, networking, monitoring) instead of duplicating them.

4. **Policy at the highest safe scope**
   - Common controls (logging, security baselines, allowed SKUs, tags) are assigned at parent MGs so that all child subscriptions inherit them.

5. **Least privilege and role separation**
   - Platform teams, workload teams, and security teams have clearly separated scopes and roles.

---

## 3. Proposed Management Group Hierarchy

### 3.1 High-level hierarchy

```text
Tenant Root Group
└─ contoso
   ├─ platform
   │  ├─ identity
   │  ├─ management
   │  └─ connectivity
   └─ landing-zones
      ├─ corp
      │  ├─ corp-prod
      │  └─ corp-nonprod
      ├─ online
      │  ├─ online-prod
      │  └─ online-nonprod
      └─ sandbox