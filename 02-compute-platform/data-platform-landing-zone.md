# Data Platform Landing Zone Architecture

**Scope:**  
Enterprise Data Platform landing zone aligned to Azure Landing Zone architecture and the governance model defined in:

- `00-foundation/management-group-design.md`
- `00-foundation/rbac-architecture.md`
- `00-foundation/policy-baseline.md`
- `01-network-architecture/hub-spoke-topology.md`

This document defines the architecture, governance integration, and operational model for deploying analytics and data workloads securely at enterprise scale.

---

## 1. Design Objectives

- Provide isolated subscriptions for data workloads (Prod / NonProd)
- Enforce Zero Trust networking using Private Endpoints
- Integrate with centralized hub-spoke network
- Apply governance guardrails via Azure Policy
- Enable secure CI/CD deployment model
- Support scalable analytics platforms (e.g., Fabric, Synapse, Databricks, SQL)

---

## 2. Subscription Placement

Data platform subscriptions are created under:

`landing-zones → corp → corp-prod`  
`landing-zones → corp → corp-nonprod`

Example subscriptions:

- `Corp-DataPlatform-Prod`
- `Corp-DataPlatform-NonProd`

This provides:

- Inherited governance controls via management group scope
- Defined blast-radius boundaries per subscription
- Independent quota and cost control
- Explicit environment-level isolation

---

## 3. Network Integration

Each Data Platform subscription contains:

- A dedicated Spoke VNet
- Subnets for:
  - `data`
  - `private-endpoints`
  - `integration`
  - `management`

The Spoke VNet is:

- Peered to the Hub VNet in the Connectivity subscription
- Configured with UDRs forcing outbound traffic via Azure Firewall
- Denied direct public internet access

All egress traffic flows:

Spoke → Azure Firewall → Internet

---

## 4. Private Endpoint Enforcement

Public network access is disabled via Azure Policy.

All PaaS services are accessed via:

- Private Endpoints
- Private DNS Zones hosted in the Hub subscription
- DNS links to spoke VNets

Services typically using Private Endpoints:

- Azure Storage
- Azure SQL
- Azure Key Vault
- Synapse / Databricks control plane endpoints (where applicable)

This enforces network-level Zero Trust by eliminating public endpoints and restricting service access to private address space.

---

## 5. RBAC Model for Data Platform

Role assignments follow the model defined in `rbac-architecture.md`.

### Data Platform Engineer Group

- Role: Contributor
- Scope: Data Platform subscription
- Responsibility: Deploy and manage analytics services

### DataOps / CI-CD Service Principal

- Role: Contributor (or custom least-privilege role)
- Scope: Resource group or subscription
- Purpose: Automated infrastructure deployments

### Security Operations

- Role: Security Reader
- Scope: Subscription
- Purpose: Monitor Defender and audit logs

### Data Consumers

- No direct Azure RBAC access required.
- Access managed within data services (e.g., Fabric workspaces, Synapse RBAC).

---

## 6. Policy Enforcement

Inherited policies from Landing Zones:

- Deny public network access
- Enforce diagnostic settings
- Require tagging
- Restrict regions
- Enforce encryption at rest

Additional data-specific policies may include:

- Require Private Endpoints for storage
- Enforce minimum TLS version
- Enforce key vault soft delete and purge protection


Policy lifecycle progression:

- Audit (impact assessment)
- Remediation
- Enforced (Deny or Modify)

---

## 7. Logging and Monitoring

All diagnostics are sent to:

Central Log Analytics workspace in `Management-Prod`.

Enabled diagnostics include:

- Storage logs
- SQL auditing logs
- Key Vault access logs
- Network flow logs
- Azure Firewall logs

Outcomes:

- Centralized telemetry aggregation
- Reduced mean time to detect (MTTD) security events
- Audit-aligned logging retention for compliance validation

---

## 8. Environment Separation

Production and NonProduction:

- Separate subscriptions
- Separate VNets
- Separate data services
- Separate CI/CD pipelines

No cross-environment peering unless explicitly required.

Strict subscription and network isolation reduces the risk of cross-environment data leakage and privilege escalation.

---

## 9. CI/CD and Infrastructure as Code

Deployment model:

- Bicep / ARM / Terraform modules
- Version-controlled repositories
- Role assignments defined as code
- Policy assignments defined as code

Pipeline stages:

1. Validate templates
2. Deploy to NonProd
3. Validate security posture
4. Promote to Prod

Infrastructure as Code enables repeatable, auditable deployments aligned with centrally enforced Azure Policy and RBAC controls.

---

## 10. Scalability Considerations

As data workloads scale:

- Separate subscriptions per data domain if required
- Dedicated VNet per domain
- Regional expansion with multiple hubs
- Consider Azure Virtual WAN for global scale

Future evolution may include:

- Data mesh architecture
- Dedicated ingestion landing zones
- Cross-tenant B2B analytics sharing

---

## 11. Alignment to Solution Architect Responsibilities

This Data Platform Landing Zone demonstrates:

- Enterprise subscription strategy
- Governance-aligned deployment
- Zero Trust data access
- Secure network integration
- Operational maturity through logging and automation

Combined with:

- Management Groups
- RBAC
- Policy baseline
- Hub-Spoke networking

This forms a complete enterprise-ready Azure architecture suitable for Microsoft Solution Architect-level discussions.