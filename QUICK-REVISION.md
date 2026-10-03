# Quick Revision

For after you've studied the domain files. One pass should take about 30 minutes.

## R1. Identities and governance

```text
Entra ID = who · RBAC = what they can do · Policy = what's allowed · Lock = can't change · Mgmt group > Sub > RG > Resource
```

- RBAC = principal + role + scope; **additive, inherited**. Contributor **can't** assign roles.
- Entra roles (users, licenses) ≠ Azure RBAC roles (resources).
- Policy: **Deny** blocks, **Audit** reports, **Modify/DeployIfNotExists** need **remediation** for existing resources.
- Locks: **CanNotDelete / ReadOnly**, inherited, apply to **Owners**. ReadOnly can block listing keys.
- Tags **not inherited**; Policy enforces/inherits.
- RG: one per resource, no nesting, RG region ≠ resource region.
- Changing a subscription's tenant **drops RBAC**.
- Management groups: **max 6 levels**, one parent per subscription.
- Budgets **alert only**. Advisor: Reliability, Security, Performance, Operational Excellence, Cost.
- Group licensing and dynamic groups = **P1+**. License needs **usage location**.

| Pair | Difference |
|---|---|
| RBAC vs Policy | Who can act vs what resources may look like |
| Lock vs Policy | Stop change/delete vs enforce standards |
| RBAC vs Entra roles | Azure resources vs directory |

## R2. Storage

```text
Firewall (network) AND authorization (RBAC / SAS / key) must both allow
```

- Redundancy: **LRS** 1 site · **ZRS** zones · **GRS** + paired region · **GZRS** zones + region · **RA-** = readable secondary. Geo copy is **async**.
- ZRS ≠ region protection. Redundancy ≠ backup.
- Tiers: Hot · **Cool 30d** · **Cold 90d** · **Archive 180d** (offline, rehydrate, **not on ZRS/GZRS**). Account default can't be Archive.
- SAS: **user delegation** (Entra), **service** (key, one service, **stored access policy** possible), **account** (key, many services, no policy). Ad hoc SAS revoke = **rotate key**.
- Keys: `key1/key2` rotate; Contributor can list keys.
- Firewall: **IP rules ignore same-region traffic**; use **service endpoint** (VNet rule) or **private endpoint**.
- Object replication: **versioning + change feed**.
- Lifecycle: moves **cooler** or deletes; can't rehydrate.
- Files: identity-based = **share RBAC + NTFS**; port **445**; share soft delete ≠ file recovery (use snapshots).
- Encryption at rest: always on; MMK or **CMK (Key Vault)**; infrastructure encryption at creation.
- AzCopy: `copy`, `sync` (one-way); auth via **Entra login or SAS**.

## R3. Compute

- ARM/Bicep: **declarative, idempotent**. **Incremental** (default) vs **Complete** (**deletes extras**). Export from portal; `az bicep decompile` ARM→Bicep.
- VM billing: **deallocated** stops compute; OS shutdown doesn't. Resize restarts.
- Move VM: RG/subscription = Move; **region = Resource Mover / Site Recovery**.
- **Encryption at host** (incl. temp disk) vs **ADE**: not together.
- **Availability set** (rack/maintenance, 99.95%) vs **zone** (datacenter, 99.99%).
- VMSS: **Flexible** recommended; autoscale min/max/default, scale-in threshold well below scale-out.
- ACR: **Premium** = geo-replication, private link. Admin user off by default.
- **ACI** = no autoscale. **Container Apps** = scale rules, scale to zero, revisions.
- App Service: **plan = compute**; tier gates features.
  - **Standard+**: slots (5), autoscale. **Basic+**: custom domain TLS binding, backup. **Free**: no custom domain.
  - Root domain = **A**; subdomain = **CNAME**; **TXT asuid** for ownership.
  - Swap = no downtime; **slot settings** stay.
  - Backup: automatic hourly/30d; custom needs **storage account**.
  - **VNet integration = outbound**; **private endpoint = inbound**.

## R4. Networking

```text
Packet path: DNS → route (UDR) → NSG (subnet then NIC) → OS firewall → app
```

- VNet regional; subnets **5 reserved IPs**. Peering **non-transitive**, no overlap.
- **NSG**: stateful, lowest priority number first. Inbound subnet→NIC; outbound NIC→subnet. Defaults: allow VNet + AzureLB, deny rest.
- **ASG** groups NICs (same VNet).
- **UDR**: associate to subnet; longest prefix; UDR > BGP > system; NVA needs **IP forwarding**.
- **Bastion**: `AzureBastionSubnet` /26, no VM public IP, TLS 443.
- **Service endpoint**: public IP, subnet-based, no on-prem. **Private endpoint**: private IP, one resource, **DNS (`privatelink`)**. **Private Link** = the technology.
- **Standard** public IP / LB = **secure by default** (NSG needed). **Basic SKU retired Sept 30, 2025.**
- New VNets: **private subnets by default** → need NAT gateway/explicit outbound.
- DNS: public zone = delegate NS; **no apex CNAME**; private zone = **link VNets**; auto-registration needs registration link.
- LB = **L4**; probe from **168.63.129.16**; Standard needs NSG and outbound rule. App Gateway = **L7 regional + WAF**. Front Door = **L7 global**.
- Internal LB **hairpin**: backend VM can't reach its own frontend.

| Pair | Difference |
|---|---|
| NSG vs Azure Firewall | Basic L3/L4 filter vs managed central firewall (FQDN, threat intel) |
| Peering vs VPN Gateway | Azure↔Azure backbone vs on-prem encrypted tunnel |
| Service vs private endpoint | Subnet identity vs private IP |

## R5. Monitoring and maintenance

- **Metrics** numeric/automatic; **Logs** need **diagnostic settings**, queried with **KQL** in **Log Analytics**.
- **Activity log** = control plane (who did what). Guest OS/memory needs **AMA + DCR**.
- Alert = **scope + condition + action group**. **No action group, no notification.** **Processing rule** suppresses/adds actions (maintenance windows).
- Alert types: metric, log search, activity log.
- Network Watcher: **IP flow verify** (NSG), **Next hop** (route), **Connection troubleshoot** (once), **Connection monitor** (continuous). NSG flow logs → **VNet flow logs**.
- Vaults: **Recovery Services** (VMs, Files, SQL/SAP, MARS, **ASR**) vs **Backup vault** (Blobs, Disks, PostgreSQL).
- Backup vault **same region as VM**. Policy = **schedule + retention**. Restore: new VM, disks, replace, file recovery, cross-region (GRS). **Soft delete 14 days**.
- ASR: vault in **target region**; **test failover** (isolated) → **failover** → **commit** → **re-protect**. Recovery plan orders VMs.

| Pair | Difference |
|---|---|
| Backup vs ASR | Restore data vs keep service running |
| Metrics vs Logs | Numbers vs records |
| Alert vs action group | Condition vs response |

## Scenario patterns

| Requirement | Think |
|---|---|
| "Who can do X?" | RBAC (or Entra role for directory tasks) |
| "Must be allowed/denied across subscriptions" | Policy at management group |
| "Prevent accidental delete" | Lock |
| "Private IP for PaaS, also from on-premises" | Private endpoint + private DNS |
| "VMs reach PaaS, restrict to subnet" | Service endpoint + firewall VNet rule |
| "RDP with no public IP" | Bastion |
| "Route by URL / WAF" | Application Gateway |
| "Non-HTTP load balancing" | Load Balancer |
| "Zero-downtime deploy, rollback" | App Service slot swap |
| "Autoscale containers, scale to zero" | Container Apps |
| "Autoscale VMs" | VMSS |
| "Survive datacenter failure" | Zones / ZRS |
| "Survive region failure" | GRS/GZRS (data), ASR (VMs) |
| "Recover deleted/corrupted data" | Backup, soft delete, versioning |
| "Notify on condition" | Alert rule + action group |
| "Suppress alerts during maintenance" | Alert processing rule |
