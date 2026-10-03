# Microsoft Resources and Coverage Map

All links are to official Microsoft sources. Skills measured as of **April 17, 2026**.

## Exam and certification

- [Certification page](https://learn.microsoft.com/en-us/credentials/certifications/azure-administrator/)
- [Exam study guide (skills measured)](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-104)
- [Free practice assessment](https://learn.microsoft.com/en-us/credentials/certifications/exams/az-104/practice/assessment?assessment-type=practice&assessmentId=21)
- [Exam sandbox](https://aka.ms/examdemo)
- [Exam scoring and score reports](https://learn.microsoft.com/en-us/credentials/certifications/exam-scoring-reports)
- [Retake policy](https://learn.microsoft.com/en-us/credentials/support/retake-policy)
- [Request accommodations](https://learn.microsoft.com/en-us/credentials/certifications/request-accommodations)
- [Renewal](https://learn.microsoft.com/en-us/credentials/certifications/renew-your-microsoft-certification)

## Microsoft Learn learning paths

The instructor-led course [AZ-104T00](https://learn.microsoft.com/en-us/training/courses/az-104t00) is built from these paths:

| Path | Matches |
|---|---|
| [Prerequisites for Azure administrators](https://learn.microsoft.com/en-us/training/paths/az-104-administrator-prerequisites/) | Background (ARM, CLI/PowerShell, templates) |
| [Manage identities and governance in Azure](https://learn.microsoft.com/en-us/training/paths/az-104-manage-identities-governance/) | [Domain 1](../01-identities-governance/README.md) |
| [Implement and manage storage in Azure](https://learn.microsoft.com/en-us/training/paths/az-104-manage-storage/) | [Domain 2](../02-storage/README.md) |
| [Deploy and manage Azure compute resources](https://learn.microsoft.com/en-us/training/paths/az-104-manage-compute-resources/) | [Domain 3](../03-compute/README.md) |
| [Configure and manage virtual networks for Azure administrators](https://learn.microsoft.com/en-us/training/paths/az-104-manage-virtual-networks/) | [Domain 4](../04-networking/README.md) |
| [Monitor and back up Azure resources](https://learn.microsoft.com/en-us/training/paths/az-104-monitor-backup-resources/) | [Domain 5](../05-monitoring-maintenance/README.md) |

## Documentation by service

| Area | Docs |
|---|---|
| Identity | [Microsoft Entra ID](https://learn.microsoft.com/en-us/entra/identity/) · [Azure RBAC](https://learn.microsoft.com/en-us/azure/role-based-access-control/overview) |
| Governance | [Azure Policy](https://learn.microsoft.com/en-us/azure/governance/policy/overview) · [Resource locks](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/lock-resources) · [Management groups](https://learn.microsoft.com/en-us/azure/governance/management-groups/overview) · [Cost Management](https://learn.microsoft.com/en-us/azure/cost-management-billing/costs/) · [Advisor](https://learn.microsoft.com/en-us/azure/advisor/) |
| Storage | [Storage documentation](https://learn.microsoft.com/en-us/azure/storage/) · [Redundancy](https://learn.microsoft.com/en-us/azure/storage/common/storage-redundancy) · [Access tiers](https://learn.microsoft.com/en-us/azure/storage/blobs/access-tiers-overview) · [SAS](https://learn.microsoft.com/en-us/azure/storage/common/storage-sas-overview) · [Storage Explorer](https://learn.microsoft.com/en-us/azure/storage/storage-explorer/vs-azure-tools-storage-manage-with-storage-explorer) |
| Compute | [ARM templates](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/overview) · [Bicep](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/overview) · [Virtual machines](https://learn.microsoft.com/en-us/azure/virtual-machines/) · [VM Scale Sets](https://learn.microsoft.com/en-us/azure/virtual-machine-scale-sets/) · [Container Registry](https://learn.microsoft.com/en-us/azure/container-registry/) · [Container Instances](https://learn.microsoft.com/en-us/azure/container-instances/) · [Container Apps](https://learn.microsoft.com/en-us/azure/container-apps/) · [App Service](https://learn.microsoft.com/en-us/azure/app-service/) |
| Networking | [Virtual Network](https://learn.microsoft.com/en-us/azure/virtual-network/) · [NSG](https://learn.microsoft.com/en-us/azure/virtual-network/network-security-groups-overview) · [Bastion](https://learn.microsoft.com/en-us/azure/bastion/) · [Private Link](https://learn.microsoft.com/en-us/azure/private-link/) · [Azure DNS](https://learn.microsoft.com/en-us/azure/dns/) · [Load Balancer](https://learn.microsoft.com/en-us/azure/load-balancer/) · [Application Gateway](https://learn.microsoft.com/en-us/azure/application-gateway/) · [Network Watcher](https://learn.microsoft.com/en-us/azure/network-watcher/) |
| Monitoring and recovery | [Azure Monitor](https://learn.microsoft.com/en-us/azure/azure-monitor/) · [Azure Backup](https://learn.microsoft.com/en-us/azure/backup/) · [Site Recovery](https://learn.microsoft.com/en-us/azure/site-recovery/) |

## Coverage map

Every official objective → where it is covered → trap → revision.

Legend: **G** = guide section, **T** = main exam trap, **R** = [QUICK-REVISION](../QUICK-REVISION.md) section.

### Domain 1: Identities and governance (20–25%)

| Official objective | Guide | Main trap | Revision |
|---|---|---|---|
| Create users and groups | [D1 §1](../01-identities-governance/README.md) | Dynamic groups can't be edited manually | R1 |
| Manage user and group properties | D1 §1 | Usage location | R1 |
| Manage licenses in Entra ID | D1 §1 | Usage location; P1 for group licensing | R1 |
| Manage external users | D1 §1 | Guest = B2B | R1 |
| Configure SSPR | D1 §1 | None/Selected/All; writeback needs P1 | R1 |
| Manage built-in Azure roles | D1 §2 | Contributor can't assign roles | R1 |
| Assign roles at different scopes | D1 §2 | Inheritance; additive | R1 |
| Interpret access assignments | D1 §2 | Inherited vs direct; Check access | R1 |
| Implement and manage Azure Policy | D1 §3 | Deny doesn't clean up; Policy ≠ RBAC | R1 |
| Configure resource locks | D1 §4 | ReadOnly breaks operations; applies to Owner | R1 |
| Apply and manage tags | D1 §5 | Not inherited | R1 |
| Manage resource groups | D1 §6 | No nesting; RG region ≠ resource region | R1 |
| Manage subscriptions | D1 §7 | Changing tenant drops RBAC | R1 |
| Manage costs: alerts, budgets, Advisor | D1 §9 | Budgets don't stop spend | R1 |
| Configure management groups | D1 §8 | Max depth 6; one parent | R1 |

### Domain 2: Storage (15–20%)

| Official objective | Guide | Main trap | Revision |
|---|---|---|---|
| Storage firewalls and virtual networks | [D2 §3](../02-storage/README.md) | IP rules don't apply same-region | R2 |
| Create and use SAS tokens | D2 §4 | Ad hoc SAS can't be revoked | R2 |
| Configure stored access policies | D2 §4 | Only service SAS; revoke handle | R2 |
| Manage access keys | D2 §4 | Rotate key1/key2; Contributor can list keys | R2 |
| Identity-based access for Azure Files | D2 §5 | Share RBAC + NTFS; port 445 | R2 |
| Create and configure storage accounts | D2 §1 | Naming; encryption always on | R2 |
| Configure storage redundancy | D2 §2 | GRS secondary not readable; ZRS ≠ region | R2 |
| Configure object replication | D2 §6 | Needs versioning + change feed | R2 |
| Configure storage account encryption | D2 §1 | CMK vs MMK; infrastructure encryption at creation | R2 |
| Storage Explorer and AzCopy | D2 §1 | `sync` vs `copy`; SAS or Entra | R2 |
| Create/configure Azure Files share | D2 §7 | Share vs file recovery | R2 |
| Create/configure Blob container | D2 §6 | Access level; anonymous disabled by default | R2 |
| Configure storage tiers | D2 §6 | Archive offline; not on ZRS | R2 |
| Soft delete for blobs and containers | D2 §6 | Not a backup of everything | R2 |
| Snapshots and soft delete for Azure Files | D2 §7 | Share soft delete ≠ file recovery | R2 |
| Blob lifecycle management | D2 §6 | Can't rehydrate | R2 |
| Blob versioning | D2 §6 | Versioning vs snapshot | R2 |

### Domain 3: Compute (20–25%)

| Official objective | Guide | Main trap | Revision |
|---|---|---|---|
| Interpret an ARM template or Bicep | [D3 §1](../03-compute/README.md) | Read params → resources → outputs | R3 |
| Modify an ARM template / Bicep | D3 §1 | Incremental vs Complete | R3 |
| Deploy with ARM/Bicep | D3 §1 | Scopes | R3 |
| Export deployment / convert ARM → Bicep | D3 §1 | Exports are not clean | R3 |
| Create a VM | D3 §2 | Deallocate vs stopped | R3 |
| Encryption at host | D3 §2 | Not with ADE | R3 |
| Move VM (RG, subscription, region) | D3 §2 | Region needs Resource Mover | R3 |
| Manage VM sizes | D3 §2 | Resize restarts | R3 |
| Manage VM disks | D3 §2 | Temp disk is non-persistent | R3 |
| Availability zones and sets | D3 §2 | Set ≠ zone | R3 |
| Deploy/configure VMSS | D3 §2 | Flexible; autoscale thresholds | R3 |
| Azure Container Registry | D3 §3 | Admin user | R3 |
| Azure Container Instances | D3 §3 | No autoscale | R3 |
| Azure Container Apps | D3 §3 | Scale rules, scale to zero | R3 |
| Container sizing and scaling | D3 §3 | ACI vs ACA | R3 |
| App Service plan | D3 §4 | Plan = compute | R3 |
| Scale an App Service plan | D3 §4 | Up vs out; autoscale Standard+ | R3 |
| Create an App Service | D3 §4 | Tier gates features | R3 |
| Certificates and TLS | D3 §4 | Binding needs Basic+ | R3 |
| Map custom DNS name | D3 §4 | A vs CNAME; TXT | R3 |
| Backup for App Service | D3 §4 | Basic+; custom needs storage | R3 |
| Networking for App Service | D3 §4 | VNet integration = outbound | R3 |
| Deployment slots | D3 §4 | Standard+; slot settings | R3 |

### Domain 4: Networking (15–20%)

| Official objective | Guide | Main trap | Revision |
|---|---|---|---|
| Create VNets and subnets | [D4 §1](../04-networking/README.md) | 5 reserved IPs; private subnet default | R4 |
| VNet peering | D4 §2 | Non-transitive | R4 |
| Public IP addresses | D4 §3 | Standard needs NSG | R4 |
| User-defined routes | D4 §4 | Associate to subnet; longest prefix | R4 |
| Troubleshoot network connectivity | D4 §10 | Tool selection | R4 |
| NSGs and ASGs | D4 §5 | Priority; both NSGs | R4 |
| Evaluate effective security rules | D4 §5 | Combined subnet + NIC | R4 |
| Azure Bastion | D4 §6 | `AzureBastionSubnet` /26 | R4 |
| Service endpoints | D4 §7 | Public IP stays | R4 |
| Private endpoints | D4 §7 | DNS | R4 |
| Azure DNS | D4 §8 | Apex CNAME; private zone link | R4 |
| Internal/public load balancer | D4 §9 | L4; Standard; NSG | R4 |
| Troubleshoot load balancing | D4 §9 | Probe 168.63.129.16 | R4 |

### Domain 5: Monitoring and maintenance (10–15%)

| Official objective | Guide | Main trap | Revision |
|---|---|---|---|
| Interpret metrics | [D5 §1](../05-monitoring-maintenance/README.md) | VM memory needs agent | R5 |
| Configure log settings | D5 §2 | Diagnostic settings required | R5 |
| Query and analyze logs | D5 §2 | KQL operators | R5 |
| Alert rules, action groups, processing rules | D5 §3 | No action group = no notification | R5 |
| Insights for VMs, storage, networks | D5 §4 | VM insights needs AMA | R5 |
| Network Watcher and Connection monitor | D5 §5 | Troubleshoot vs monitor | R5 |
| Create a Recovery Services vault | D5 §6 | Same region as VM | R5 |
| Create an Azure Backup vault | D5 §6 | Which workloads | R5 |
| Backup policy | D5 §6 | Schedule + retention | R5 |
| Backup and restore operations | D5 §6 | Restore options | R5 |
| Site Recovery configuration | D5 §7 | Vault in target region | R5 |
| Failover with Site Recovery | D5 §7 | Test failover first; re-protect | R5 |
| Backup reports and alerts | D5 §6 | Needs diagnostic settings | R5 |
