# Hub-Spoke Network Architecture

**Scope:**  
Enterprise network topology aligned to Azure Landing Zone architecture and the governance model defined in:

- `00-foundation/management-group-design.md`
- `00-foundation/rbac-architecture.md`
- `00-foundation/policy-baseline.md`

This document defines the hub-spoke model used to provide secure connectivity, centralized egress, and controlled communication between landing zones.

---

## 1. Design Objectives

- Centralize shared services (firewall, DNS, VPN, Bastion)
- Enforce secure egress inspection
- Isolate workloads across subscriptions
- Support hybrid connectivity
- Enable scalable landing zone onboarding

---

## 2. High-Level Topology

```text
On-Premises Network
        │
        │ (VPN / ExpressRoute)
        │
   ┌───────────────┐
   │  Hub VNet     │  (Connectivity Subscription)
   │               │
   │  - Azure Firewall
   │  - VPN / ER Gateway
   │  - Bastion
   │  - Private DNS
   └───────────────┘
        │
        ├───────────────┬───────────────┬───────────────
        │               │               │
  Spoke VNet       Spoke VNet      Spoke VNet
  (Corp-Prod)      (Corp-NonProd)  (Online-Prod)
  (Workload Sub)   (Workload Sub)  (Workload Sub)

```

  ## 3. Hub VNet Design

### Components

The Hub VNet resides in the `Connectivity-Prod` subscription under the `platform` management group.

Core components:

- Azure Firewall (Premium recommended for TLS inspection and IDPS)
- VPN Gateway or ExpressRoute Gateway
- Azure Bastion
- Private DNS Zones
- Centralized diagnostics configuration

### Subnet Layout

```text
AzureFirewallSubnet
GatewaySubnet
AzureBastionSubnet
SharedServicesSubnet

```


---

## 4. Spoke VNet Design

Each workload subscription contains its own Spoke VNet.

Typical spoke subnet structure:

```text
app-subnet
data-subnet
private-endpoints-subnet
management-subnet
```

---

## 5. Secure Egress Pattern

All outbound traffic from spoke VNets follows this pattern:

1. UDR routes traffic to Azure Firewall.
2. Azure Firewall inspects traffic.
3. Traffic is allowed or denied based on rule collections.
4. Logs are sent to central Log Analytics.

Firewall rule layers:

- Network rules (IP-based filtering)
- Application rules (FQDN-based filtering)
- DNAT rules (if required)

Benefits:

- Centralized visibility
- Reduced attack surface
- Compliance enforcement
- Controlled outbound access

## 6. Private Endpoint Strategy

Public network access is disabled for PaaS services via Azure Policy.

All critical services (Storage, SQL, Key Vault, etc.) are accessed through:

- Private Endpoints
- Private DNS Zones hosted in the Hub subscription
- DNS zone links to spoke VNets

Design benefits:

- Zero Trust alignment
- No exposure to public internet
- Controlled service-to-service communication
- Policy-enforced compliance


## 7. Hybrid Connectivity

Hybrid connectivity is supported through:

- ExpressRoute (preferred for production)
- VPN Gateway (acceptable for smaller or dev environments)

Traffic flow model:

On-prem → Gateway in Hub → Firewall (if required) → Spoke

Route propagation is controlled via:

- BGP configuration
- Custom route tables where segmentation is required

All hybrid traffic traverses the hub for inspection and logging.


## 8. Network Security Controls

Security enforcement layers include:

- NSGs applied at subnet level
- Azure Firewall rule collections
- Application rules for FQDN filtering
- Deny-by-default outbound configuration
- DDoS Protection Plan (if public exposure exists)

No workload subscription exposes direct internet ingress without:

- Azure Firewall
- Application Gateway with WAF
- Azure Front Door (if applicable)

This enforces layered defense.


## 9. Logging and Monitoring

Diagnostics are enforced via Azure Policy:

- NSG Flow Logs
- Azure Firewall logs
- VPN / ExpressRoute logs
- VNet peering metrics

All logs are sent to:

Central Log Analytics workspace in `Management-Prod` subscription.

Benefits:

- Unified network observability
- Incident response readiness
- Compliance audit capability


## 10. Scalability Considerations

As landing zones scale:

- Additional spokes are added.
- Hub firewall SKU can be upgraded.
- Hub segmentation may be introduced (Prod Hub / NonProd Hub).

For multi-region scale:

- Regional hubs per geography
- Azure Virtual WAN may replace traditional hub-spoke
- Cross-region peering with global reach

This design supports incremental enterprise growth.


## 11. Alignment to Solution Architect Responsibilities

This hub-spoke design demonstrates:

- Enterprise network segmentation
- Centralized security enforcement
- Hybrid-ready architecture
- Zero Trust alignment
- Governance integration with Policy and RBAC

Combined with:

- Management Group hierarchy
- RBAC architecture
- Policy baseline

This forms a complete enterprise control plane and data plane foundation suitable for Microsoft Solution Architect-level discussions.