# Monitoring and Operational Excellence Architecture

**Scope:**  
Enterprise monitoring and operational model aligned with:

- `00-foundation/management-group-design.md`
- `00-foundation/rbac-architecture.md`
- `00-foundation/policy-baseline.md`
- `01-network-architecture/hub-spoke-topology.md`
- `02-compute-platform/data-platform-landing-zone.md`
- `02-compute-platform/ha-web-tier-vmss.md`
- `03-storage-backup-dr/storage-backup-dr-architecture.md`

This document defines how the platform is observed, how incidents are detected and handled, and how operational health is managed across the Azure estate.

---

## 1. Design Objectives

- Provide consistent observability across all subscriptions and workloads
- Detect and respond to incidents quickly and predictably
- Align monitoring with governance, security and compliance requirements
- Enable teams to self-serve dashboards while maintaining central visibility
- Control monitoring cost without sacrificing critical coverage

Monitoring design follows Azure Well-Architected Framework pillars, especially Reliability, Operational Excellence and Cost Optimization.

---

## 2. Log and Metric Architecture

A central monitoring topology is used:

- One or more **central Log Analytics workspaces** in the management subscription for platform-wide logs
- Workload-specific workspaces where application-team isolation is required
- Diagnostics routed according to policy baseline:
  - Activity Logs from all subscriptions
  - NSG flow logs
  - Azure Firewall and gateway logs
  - Storage, Key Vault, database and other PaaS diagnostics
  - VM and VMSS guest diagnostics where applicable

Metrics are collected using:

- Native Azure metrics for platform signals (CPU, memory, network, disk, availability)
- Custom metrics where needed for application-specific SLIs

Workspace design is documented to avoid excessive fragmentation while respecting data residency and organizational boundaries.

---

## 3. Diagnostic Settings and Policy Integration

Diagnostic settings are not configured manually per resource; they are enforced via Azure Policy:

- Policy assignments at management group level ensure:
  - New resources automatically send diagnostics to the correct workspaces
  - Required categories (e.g., Audit, Security, Request, Error) are enabled
  - Retention periods are applied consistently

This removes configuration drift risk and ensures that new subscriptions and workloads inherit the same monitoring baseline.

---

## 4. Alerting Strategy

Alerts are structured into three layers:

1. **Platform Alerts**  
   - Subscription-level and regional issues (e.g., service health, quota, firewall health)
   - Triggered from central workspaces and Azure Monitor metrics

2. **Workload Infrastructure Alerts**  
   - VMSS availability and health probe failures  
   - CPU/memory saturation beyond thresholds  
   - Storage capacity thresholds and high latency  
   - Backup success/failure notifications

3. **Application-Level Alerts**  
   - HTTP error rate (4xx/5xx) on front-end endpoints  
   - Request latency and timeouts  
   - Custom business KPIs tied to SLIs/SLOs

Alert rules are defined as templates where possible to maintain consistency across environments.

Action groups route alerts to:

- Operations teams (email, Teams, ITSM integration)
- On-call engineers (SMS/phone where required)
- Automation runbooks for auto-remediation scenarios

---

## 5. Incident Management and Runbooks

Incident handling is standardized:

- Alerts are integrated with an ITSM tool or ticketing system where available.
- Each critical alert has an associated **runbook** describing:
  - How to triage the issue
  - What data to collect (logs, metrics, traces)
  - Initial mitigation steps
  - Escalation paths and ownership

Examples:

- VMSS capacity issues: scale configuration review and temporary manual scale-out
- Firewall saturation: rule optimization, temporary capacity increase
- Storage account throttling: workload investigation and tiering adjustments

Runbooks are stored alongside infrastructure as code or in a central documentation repository and reviewed periodically.

---

## 6. Dashboards and Reporting

Dashboards are created for different audiences:

- **Platform dashboard**  
  - Subscription health, resource deployment trends, policy compliance, cost overview

- **Network and security dashboard**  
  - Firewall and NSG activity, VPN/ExpressRoute status, secure score

- **Workload dashboards**  
  - VMSS instance health, web/API availability, database performance, key storage metrics

Tools:

- Azure Monitor workbooks for interactive analysis
- Azure Dashboards for high-level overviews
- Power BI for aggregated reporting across subscriptions and time periods

Dashboards are shared using RBAC so teams can view relevant information without excessive permissions.

---

## 7. Reliability, SLOs and Continuous Improvement

For critical services, Service Level Objectives (SLOs) are defined:

- Availability targets for front-end endpoints
- Performance targets (latency, throughput)
- Recovery time and recovery point targets (RTO/RPO) aligned with backup/DR design

Error budgets and trend analysis are used to:

- Identify recurring reliability issues
- Prioritize engineering work to remove chronic sources of incidents
- Feed back into architecture decisions (e.g., scaling, redundancy, regional deployment)

Post-incident reviews are used to update runbooks, alerts and dashboards.

---

## 8. Cost Monitoring and Optimization

Monitoring itself can be a significant cost driver; cost management is integrated:

- Log ingestion and retention settings are tuned per data type and importance.
- High-volume, low-value logs are sampled or excluded where safe.
- Regular reviews identify unused alerts, noisy rules and inactive dashboards.
- Cost reports for Log Analytics and Azure Monitor are part of monthly governance reviews.

This ensures observability remains effective without uncontrolled cost growth.

---

## 9. Security and Compliance Considerations

Monitoring data often contains sensitive information. Controls include:

- RBAC-restricted access to Log Analytics workspaces
- Separation of duties between security, operations and development teams
- Data retention policies aligned with regulatory requirements
- Use of Microsoft Sentinel or equivalent SIEM (where adopted) on top of central workspaces for advanced threat detection

Audit requirements are supported through:

- Immutable logging for key systems where required
- Evidence of backup success, DR tests and access reviews

---

## 10. Solution Architect Alignment

This monitoring and operational excellence architecture demonstrates:

- End-to-end observability across governance, network, compute, storage and data workloads
- Integration of monitoring with policy, RBAC and security posture
- A structured approach to alerting, incident response and continuous improvement
- Awareness of cost and compliance implications of observability

Combined with the foundation, network, compute, storage/DR and data platform designs, it completes a cohesive Azure infrastructure architecture portfolio appropriate for Solution Architect-level discussions.