# Azure RBAC Architecture and Role Model

**Scope:**  
Authorization model for an enterprise Azure environment aligned to Azure Landing Zone architecture, aligned to the management group hierarchy defined in `management-group-design.md`.

This RBAC design aligns with Azure Landing Zone and Microsoft Cloud Adoption Framework (CAF) guidance, specifically within the design areas of:

- Identity and access management
- Resource organization
- Governance and compliance


This document describes **how access is structured**, which personas exist, where roles are granted, and how **least-privilege** and **separation of duties** are enforced across platform and landing zones.

---

## 1. Design Principles

1. **Least privilege by default**  
   Every role assignment has:
   - The lowest possible permission level
   - The narrowest possible scope (management group → subscription → resource group → resource)

2. **Separation of duties**  
   Platform, security, and workload teams do not share the same high-privilege roles or scopes.

3. **Management group first, not RBAC chaos**  
   Role assignments follow the **management group hierarchy**, avoiding ad-hoc subscription-level sprawl.

4. **Just-in-time elevation with PIM**  
   High-impact roles (Owner, User Access Administrator, Security Admin) are granted through **Microsoft Entra Privileged Identity Management (PIM)** with time-bound activation.

5. **Consistency across environments**  
   Access patterns are the same for `prod` and `nonprod` – only the number of people and approval strictness change, not the model.

---

## 2. Key Personas

| Persona                     | Description                                                |
|----------------------------|------------------------------------------------------------|
| **Cloud Platform Owner**   | Owns overall Azure platform; manages MG structure & policy |
| **Platform Engineer**      | Deploys & operates shared services (identity, connectivity, management) |
| **Security Operations**    | Monitors security posture, Defender, Sentinel, audit logs  |
| **Data Platform Engineer** | Owns data platform subscriptions and workloads             |
| **App/Product Team**       | Owns specific application workloads in landing zones       |
| **Finance/Cost Analyst**   | Monitors cost and usage, no configuration rights          |
| **Sandbox User**           | Developer/engineer experimenting in sandbox subscriptions  |

These personas map to Entra security groups which are then used for RBAC assignments.

---

## 3. Role Assignment Strategy by Scope

### 3.1 Tenant Root and `contoso` Management Group

**Scope:** `Tenant Root Group` and `contoso` (top-level MG below tenant root)

- **Cloud Platform Owner**
  - Role: `Owner` (PIM-protected)
  - Scope: `contoso` MG
  - Purpose: Maintain MG structure, high-level policy assignments, cross-subscription governance.

- **Security Operations**
  - Role: `Security Reader`
  - Scope: `contoso` MG
  - Purpose: Read access to security posture across all child MGs and subscriptions.

- **Cost / Finance**
  - Role: `Cost Management Reader`
  - Scope: `contoso` MG
  - Purpose: Cross-org cost visibility without configuration rights.

> **Rule:** No permanent `Owner` or `User Access Administrator` assignments at Tenant Root. All such roles are PIM-based and audited.
All role assignments at management group scope are group-based and reviewed quarterly as part of governance controls.

---

### 3.2 Platform Management Groups (`platform`, `identity`, `management`, `connectivity`)

**Scope:** `platform` MG and its child MGs.

- **Platform Owner Group**
  - Role: `Owner`
  - Scope: `platform` MG
  - Purpose: Full control over platform subscriptions.

- **Platform Engineer Group**
  - Role: `Contributor`
  - Scope: individual platform subscriptions (e.g., `Connectivity-Prod`, `Management-Prod`)
  - Purpose: Deploy and manage shared network, logging, and identity infra.

- **Security Operations**
  - Roles:
    - `Security Admin` at `platform` MG for security configuration
    - `Log Analytics Contributor` on management subscriptions if needed
  - Scope: `platform` MG + specific resource groups (e.g., central Log Analytics workspace)

> **Pattern:** Platform team controls platform resources; workload teams never get `Contributor` on platform subscriptions.

---

### 3.3 Landing Zone Management Groups (`landing-zones`, `corp`, `online`, `sandbox`)

**Scope:** `landing-zones` MG and its children.

- **Landing Zone Owner Group**
  - Role: `Owner` (PIM)
  - Scope: `landing-zones` MG
  - Purpose: Govern landing zone structure and enforce guardrails.

- **Data Platform Engineer Group**
  - Role: `Contributor`
  - Scope: `Corp-DataPlatform-Prod`, `Corp-DataPlatform-NonProd` subscriptions
  - Purpose: Deploy and manage data platform services (Fabric, Synapse, Databricks, SQL, storage).

- **App/Product Team Groups**
  - Role: `Contributor`
  - Scope: individual workload subscriptions (e.g., `Online-Web-Prod`, `Online-Web-NonProd`)
  - Purpose: Deploy and manage app services, VMs, storage within their subscription.

- **Security Operations**
  - Role: `Security Reader` and `Log Analytics Reader`
  - Scope: `landing-zones` MG
  - Purpose: Read-only visibility of security posture and logs across all landing zones.

- **Sandbox Users**
  - Role: `Contributor`
  - Scope: Sandbox subscription(s) only
  - Additional guardrails enforced via policy rather than RBAC.

---

## 4. Resource Group and Resource-Level Access

Within each workload subscription:

1. **Standard structure**

   ```text
   rg-network        # vNet, NSGs, firewall rules
   rg-compute        # VMs, VMSS, scale sets, availability sets
   rg-data           # Storage, databases, key vaults
   rg-monitoring     # Alerts, diagnostic settings, workbooks