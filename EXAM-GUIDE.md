# Exam Guide: AZ-104

## Official facts (Microsoft)

Source: [AZ-104 study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-104) and [certification page](https://learn.microsoft.com/en-us/credentials/certifications/azure-administrator/). Skills measured as of **April 17, 2026**.

| Item | Detail |
|---|---|
| Certification | Microsoft Certified: Azure Administrator Associate |
| Level | Intermediate |
| Duration | 100 minutes |
| Passing score | 700 or greater |
| Delivery | Proctored through Pearson VUE |
| Question types | Can include interactive components |
| Retake | After 24 hours for the first retake; longer waits after that ([retake policy](https://learn.microsoft.com/en-us/credentials/support/retake-policy)) |
| Renewal | Every 12 months, free online assessment on Microsoft Learn |
| Languages | English, Chinese (Simplified), Korean, Japanese, French, Spanish, German, Portuguese (Brazil), Chinese (Traditional), Italian |
| Preview features | Most questions cover GA features; commonly used preview features may appear |
| Note from Microsoft | The bullets under each skill illustrate how it is assessed; related topics may also appear |

Microsoft does not publish the number of questions. Do not trust sites that claim a fixed number.

## Audience and experience (official)

You implement, manage and monitor an Azure environment: networks, storage, compute, identity, security, governance. You work with other roles (networking, security, database, developers, DevOps).

You should be familiar with operating systems, networking, servers and virtualization, and have experience with the Azure portal, PowerShell, Azure CLI, ARM templates or Bicep, and Microsoft Entra ID.

## Domains and weights (official)

| # | Domain | Weight | Guide |
|---|---|---|---|
| 1 | Manage Azure identities and governance | 20–25% | [01](01-identities-governance/README.md) |
| 2 | Implement and manage storage | 15–20% | [02](02-storage/README.md) |
| 3 | Deploy and manage Azure compute resources | 20–25% | [03](03-compute/README.md) |
| 4 | Implement and manage virtual networking | 15–20% | [04](04-networking/README.md) |
| 5 | Monitor and maintain Azure resources | 10–15% | [05](05-monitoring-maintenance/README.md) |

Identity and compute together are up to half of the exam. Networking is the area where most candidates lose points.

## Important services (derived from the objectives)

Microsoft Entra ID, Azure RBAC, Azure Policy, resource locks, tags, management groups, Cost Management, Advisor · Storage accounts, Blob, Azure Files, SAS, AzCopy · ARM/Bicep, VMs, disks, scale sets, ACR, ACI, Container Apps, App Service · VNet, peering, UDR, NSG/ASG, Bastion, service/private endpoints, Azure DNS, Load Balancer · Azure Monitor, Log Analytics, alerts, Network Watcher, Azure Backup, Site Recovery.

## Recommended preparation (our advice, not Microsoft's)

1. Read each domain file once, slowly. Do the Quick Checks.
2. Spend extra time on comparison tables. Most wrong answers come from mixing up similar services.
3. If you can, click through each service once in the portal. Microsoft recommends hands-on experience, even though this repo has no labs.
4. Take the free Microsoft practice assessment, then read the explanations for every miss.
5. Revise with [QUICK-REVISION.md](QUICK-REVISION.md).
6. Finish with [EXAM-DAY-CHECKLIST.md](EXAM-DAY-CHECKLIST.md).

## Official Microsoft resources

See [references/microsoft-resources.md](references/microsoft-resources.md).
