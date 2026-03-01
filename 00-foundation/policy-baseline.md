# Azure Policy Baseline Architecture

**Scope:**  
Enterprise policy framework aligned to the management group hierarchy and RBAC model defined in:

- `management-group-design.md`
- `rbac-architecture.md`

This document defines how Azure Policy is structured, assigned, and enforced across platform and landing zones to ensure governance, security, and compliance at scale.

This design aligns with Azure Landing Zone and Microsoft Cloud Adoption Framework (CAF) guidance for governance and compliance.

---

## 1. Design Principles

1. **Policy at the highest safe scope**  
   Assign policies at management group level wherever possible to ensure inheritance and consistency.

2. **Initiative-first model**  
   Policies are grouped into initiatives (policy sets) to ensure versioned, structured governance.

3. **Guardrails over manual enforcement**  
   Where possible, use:
   - `DeployIfNotExists`
   - `Deny`
   - `Modify`

   instead of relying on human processes.

4. **Separation of Platform vs Workload policy**  
   Platform subscriptions and Landing Zone subscriptions have different enforcement levels.

5. **Audit → Enforce progression**  
   New policies are deployed in `Audit` mode before being transitioned to `Deny`.

---

## 2. Policy Hierarchy by Scope

### 2.1 Tenant Root Group

Minimal policies only:

- Deny classic (ASM) resources
- Allowed Azure regions (if mandated by regulation)

These are global constraints.

---

### 2.2 `contoso` Management Group

Enterprise-wide baseline initiative:

**Initiative: `Enterprise-Global-Baseline`**

Includes:

- Required tags:
  - `owner`
  - `environment`
  - `costCenter`
- Enforce HTTPS only for App Services
- Require resource diagnostic settings
- Restrict public IP creation (audit or deny depending on environment)
- Enforce Azure Defender plan enablement

Effect types used:

- `Deny` (for critical non-compliant resources)
- `DeployIfNotExists` (for diagnostics)
- `Modify` (for auto-tagging where appropriate)

---

### 2.3 Platform Management Groups (`platform`)

Stricter policies for shared services.

**Initiative: `Platform-Foundation-Baseline`**

Examples:

- Require Azure Firewall SKU = Premium (if applicable)
- Restrict VPN Gateway SKUs
- Deny public access for storage accounts
- Enforce Private Endpoints where required
- Enforce Key Vault soft delete and purge protection

Platform policies prioritize security and stability over flexibility.

---

### 2.4 Landing Zones Management Group (`landing-zones`)

**Initiative: `LandingZone-Security-Baseline`**

Examples:

- Require managed disks for VMs
- Enforce VM backup enabled
- Require encryption at rest
- Require Log Analytics agent / AMA
- Deny public network access for:
  - Storage
  - SQL
  - Data platform services
- Restrict allowed VM SKUs
- Restrict regions (if applicable)

Landing zones focus on security and workload guardrails.

---

### 2.5 Sandbox Management Group

More relaxed, but cost-controlled.

**Initiative: `Sandbox-Guardrails`**

Examples:

- Restrict expensive VM SKUs
- Auto-shutdown policy
- Budget alerts
- Audit-only security policies (not deny)

---

## 3. Diagnostic and Logging Enforcement

All subscriptions must:

- Send Activity Logs to central Log Analytics workspace
- Enable diagnostic settings for:
  - Key Vault
  - Storage
  - SQL
  - Azure Firewall
  - NSGs

Implementation pattern:

- `DeployIfNotExists` policy to configure diagnostics automatically.
- Workspace ID passed as initiative parameter.

This ensures:

- Centralized monitoring
- Incident response capability
- Audit trail consistency

---

## 4. Security Baseline Integration

Integration with Microsoft Defender for Cloud:

- Defender plans enabled at management group scope.
- Secure score monitored centrally.
- Policies mapped to:
  - Regulatory compliance initiatives (e.g., ISO, NIST if required).

Zero Trust principles applied:

- Deny public access where possible.
- Require private endpoints.
- Enforce encryption.
- Enforce identity-based access.

---

## 5. Policy Lifecycle Management

Policy changes follow structured governance:

1. New policy introduced in `Audit` mode.
2. Compliance impact assessed.
3. Remediation tasks created if needed.
4. Policy moved to `Deny` or `Modify` once validated.

All initiatives are versioned and documented.

---

## 6. Policy as Code

Policies and initiatives should be deployed using:

- Bicep
- ARM templates
- Terraform

Benefits:

- Version control
- Change tracking
- Consistency across environments
- Automated deployment in CI/CD

Example components:

- Policy definitions
- Initiative definitions
- Assignment templates
- Parameter files

---

## 7. Relationship to Solution Architect Responsibilities

This policy baseline demonstrates:

- Governance enforcement at scale
- Control plane design maturity
- Alignment to Azure Landing Zone architecture
- Ability to design for compliance and enterprise operations

Combined with:

- Management Group architecture
- RBAC model
- Networking and landing zone design

This forms a complete enterprise control plane foundation.