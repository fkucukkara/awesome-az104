# Domain 3: Deploy and Manage Azure Compute Resources (20–25%)

Official skills: ARM templates / Bicep · virtual machines · containers · App Service.

Mental model:

```text
Declare it  → ARM / Bicep       (what should exist)
Run it      → VM / VMSS         (you manage the OS)
Package it  → Container         (ACR stores, ACI/ACA runs)
Host it     → App Service       (platform runs your web app)
```

## Know This

- **ARM template** = JSON. **Bicep** = cleaner language that compiles to ARM. Both are **declarative** and **idempotent**.
- Deployment modes: **Incremental** (default, leaves other resources) vs **Complete** (**deletes** resources in the RG that aren't in the template).
- **Availability set** = protects inside a datacenter (fault + update domains). **Availability zone** = separate datacenters in a region.
- VM SLAs: single VM with premium SSD **99.9%**, availability set **99.95%**, 2+ VMs across zones **99.99%**.
- **Stop from the OS ≠ deallocate.** Only **Deallocated** stops compute billing.
- **Encryption at host** encrypts temp disk and caches; **ADE** uses BitLocker/DM-Crypt; they can't be combined.
- **VMSS Flexible** is the recommended orchestration mode. Autoscale = metric/schedule rules, min/max/default instance count.
- **ACI** = single container group, no built-in scaling. **Container Apps** = serverless, revisions, scale rules, scale to zero.
- **App Service plan** = the compute (region, OS, size, instances). The **tier** decides features: **slots need Standard+**, backup needs **Basic+**, autoscale needs **Standard+**.
- **Scale up** = bigger tier/size. **Scale out** = more instances.

## Understand This

- How to **read** a template: parameters → variables → resources → outputs, and what `dependsOn` or a symbolic reference means.
- Why a VM resize can force a restart and why a region move isn't a simple "move".
- Why App Service plan and app are separate objects and why scaling the plan scales all apps in it.

---

## 1. ARM templates and Bicep

### What is it?

Infrastructure as code for Azure Resource Manager. Describe the **desired state**; ARM creates or updates to match.

### Know for AZ-104

- **ARM template sections:** `$schema`, `contentVersion`, `parameters`, `variables`, `resources`, `outputs` (and `functions`).
- **Bicep:** `param`, `var`, `resource`, `module`, `output`. Compiles to ARM JSON. No state file.
- **Idempotent:** deploying the same template again leaves the result unchanged.
- **Deployment scopes:** resource group, subscription, management group, tenant (Bicep `targetScope`).
- **Modify** an existing template: change a property (for example VM size or a tag), redeploy in **Incremental** mode.
- **Export:** export a deployment, a resource group or a resource as an ARM template from the portal. The export may not include everything (some properties/resources are missing or hard-coded).
- **Convert:** ARM → Bicep with decompile (`az bicep decompile`); the result is best-effort and may need fixes.
- Parameters can come from a **parameter file**. `what-if` previews changes.

Read this Bicep (exam-style interpretation):

```bicep
param location string = resourceGroup().location
param storageName string

resource sa 'Microsoft.Storage/storageAccounts@2023-05-01' = {
  name: storageName
  location: location
  sku: { name: 'Standard_GRS' }
  kind: 'StorageV2'
}

output blobEndpoint string = sa.properties.primaryEndpoints.blob
```

Reading it: deploys a **GPv2, GRS** account in the RG's region; the name is supplied at deploy time; it outputs the blob endpoint.

### Common confusion

| Question | Answer |
|---|---|
| ARM vs Bicep? | Same engine. Bicep is a nicer syntax that compiles to ARM. |
| Incremental vs Complete? | Complete **removes** resources not in the template. |
| Template vs Template spec? | A template spec is a stored, shareable, versioned template in Azure. |

> [!WARNING]
> **Exam Trap: Complete mode deletes**
>
> Deploying in Complete mode to an RG removes anything not in the template. It's rarely what you want.

> [!WARNING]
> **Exam Trap: exported templates**
>
> Exported templates are a starting point, not a guaranteed clean copy. Review parameters and secrets.

### Remember

- Declarative + idempotent. Incremental default. Export from portal; decompile to Bicep.

---

## 2. Virtual machines

### What is it?

IaaS compute. You manage the OS and everything above; Azure manages the hardware.

### Know for AZ-104

**Create:** subscription, RG, name, region, availability option, image, size, admin credentials, disks, networking, management, monitoring.

**Sizes (resize):**
- Change size from the VM's **Size** blade.
- A resize **restarts** the VM. If the new size isn't available on the current hardware cluster, **stop (deallocate)** the VM first.
- **Temp disk** data can be lost when VMs are moved to other hardware.

**Power states and billing:**

| State | Compute billed? | Dynamic public IP kept? |
|---|---|---|
| Running | Yes | Yes |
| **Stopped** (shut down inside the OS) | **Yes** | Yes |
| **Stopped (deallocated)** | **No** | No (Standard static IP stays) |

**Disks:**

| Disk | Notes |
|---|---|
| OS disk | Required; managed disk |
| Data disks | Optional; attach many |
| Temp disk | Local, **non-persistent** (often D:) |

Managed disk types: **Standard HDD, Standard SSD, Premium SSD, Premium SSD v2, Ultra**. Disk size/type can be changed (VM usually must be deallocated).

**Encryption:**

| Option | What |
|---|---|
| Server-side encryption (SSE), platform keys | **Default**, always on |
| SSE with customer-managed key | Key in Key Vault via a **Disk Encryption Set** |
| **Encryption at host** | End-to-end: encrypts temp disk + caches at the VM host; needs the feature enabled on the subscription and a supported size |
| **Azure Disk Encryption (ADE)** | BitLocker/DM-Crypt inside the OS, keys in Key Vault |

**Move a VM:**
- **Resource group / subscription:** *Move* (VM plus dependent resources like NIC, disk, public IP). Resource IDs change.
- **Region:** use **Azure Resource Mover** (or Site Recovery). There's no simple "change region".
- **Availability zones:** done via Resource Mover or recreating from a snapshot/disk.

### Availability

| | Availability set | Availability zone |
|---|---|---|
| Protects against | Rack/host failure and planned maintenance **within a datacenter** | **Datacenter** failure |
| Built from | Fault domains + update domains | Physically separate locations in a region |
| SLA (2+ VMs) | 99.95% | 99.99% |
| Use when | Region has no zones, or legacy | Region supports zones |

Plan it **at deployment**. Don't expect to flip it later.

### VM Scale Sets (VMSS)

- Group of identical (or Flexible: mixed) VMs managed as one with **autoscale**.
- **Orchestration modes:** **Flexible** (recommended; spreads across zones/fault domains, works with mixed sizes) and Uniform (older, identical instances).
- **Autoscale:** metric rules (for example CPU > 70% → +1) or **schedules**. Set **min, max, default** instances. Use a **cooldown** and make scale-in thresholds clearly lower than scale-out to avoid flapping.
- Behind a **load balancer**/App Gateway for traffic.

### Common confusion: VMSS vs individual VMs

| Question | VMSS | Individual VMs |
|---|---|---|
| Purpose | Many identical instances with autoscale | A specific VM with its own identity |
| Scaling | Automatic | Manual |
| Use | Stateless, scalable tier | Pets, singletons |
| Trap | Don't hand-configure instances; update the model | Don't rely on one VM for HA |

> [!WARNING]
> **Exam Trap: "Stopped" still bills**
>
> Shutting down inside the OS leaves the VM allocated, so you're still billed. Use **Stop** in Azure to deallocate.

> [!WARNING]
> **Exam Trap: encryption at host vs ADE**
>
> Encryption at host covers the **temp disk and caches**; ADE doesn't encrypt at the host layer. They are mutually exclusive.

> [!WARNING]
> **Exam Trap: moving across regions**
>
> *Move* in the portal works for **resource group/subscription**. A cross-region move needs Resource Mover or Site Recovery.

> [!WARNING]
> **Exam Trap: availability set ≠ zone**
>
> An availability set doesn't protect against a datacenter outage. Zones do.

### Remember

- Deallocate to stop billing. Resize restarts. Zones > sets. VMSS Flexible + autoscale.

---

## 3. Containers

### What is it?

Package an app and its dependencies in an image. **ACR** stores images; **ACI** or **Container Apps** runs them.

### Azure Container Registry (ACR)

- Private registry for images and artifacts. Name is **globally unique**.
- **SKUs:** Basic, Standard, **Premium** (geo-replication, private link, zone redundancy, content trust).
- Authentication: **Entra ID/RBAC** (`AcrPull`, `AcrPush`), **admin user** (off by default; avoid), tokens.
- Typical flow: build image → tag with the registry login server → push → pull from ACI/ACA/AKS.

### ACI vs Container Apps

| Question | Azure Container Instances | Azure Container Apps |
|---|---|---|
| What is it? | Run one or a group of containers, **no orchestration** | Serverless platform for microservices and jobs |
| Scaling | **None built in**; to scale, create more instances | **Scale rules** (HTTP, TCP, queue/event KEDA); min–max replicas; **scale to zero** |
| Resources | CPU/memory set per container group at creation | CPU/memory per replica; replicas scale out |
| Features | Container groups (shared lifecycle/network), restart policy | Revisions, traffic splitting, ingress, environments |
| Use | Simple/burst/short-lived tasks | Always-on APIs, event-driven workers |
| Trap | "Scale ACI" means more instances, not autoscale | Needs an **environment** |

- **ACI restart policy:** Always, OnFailure, Never.
- **Container Apps ingress:** external (internet) or internal (environment only).

> [!WARNING]
> **Exam Trap: ACI doesn't autoscale**
>
> If a question needs autoscaling or scale-to-zero containers, the answer is **Container Apps**, not ACI.

> [!WARNING]
> **Exam Trap: ACR admin user**
>
> The admin account is a shared credential. For production use RBAC or managed identity.

### Remember

- ACR stores. ACI = simple, no scale. ACA = scale rules, revisions, scale to zero.

---

## 4. Azure App Service

### What is it?

Managed web hosting (web apps, APIs). You deploy code or a container; Azure runs and patches the platform.

```text
App Service plan   =  the compute (region, OS, instances, size, tier)
   └── App(s)      =  your web apps, running on the plan's instances
        └── Slots  =  staging copies of an app (same plan)
```

### Know for AZ-104

**Plan and tier:**

| Tier family | Notes |
|---|---|
| Free / Shared | Shared compute, **no scale out**, limited features; **Free can't use a custom domain** |
| Basic | Dedicated; manual scale; custom domain + TLS; backup |
| **Standard** and up | **Autoscale, deployment slots (Standard: 5)**, backup |
| Premium | More scale/slots/performance |
| Isolated | Dedicated environment (ASE) |

**Scaling:**
- **Scale up** = change tier/size (more CPU/RAM/features).
- **Scale out** = more instances. Manual, or **autoscale** (Standard+) by metric or schedule.
- All apps (and slots) in a plan **share** its instances and scale together.

**Custom domain and TLS:**
- **Root domain** → **A record**. **Subdomain** → **CNAME** to `<app>.azurewebsites.net`. Add a **TXT** (`asuid`) record to prove ownership.
- Custom domains must be in **public DNS**. Free tier can't use them; TLS bindings need **Basic+**.
- **Certificates:** free **App Service managed certificate** (no wildcard, not exportable), App Service certificate, **Key Vault** import, or upload a PFX.
- **SNI SSL** (free, many certs per IP) vs **IP SSL** (dedicated IP, Standard+).
- Enforce **HTTPS Only** and a minimum TLS version.

**Backup:**
- Supported in **Basic, Standard, Premium, Isolated**. (In Basic only the production slot.)
- **Automatic:** hourly, retained 30 days, no config, not downloadable.
- **Custom:** you configure a **storage account** (SAS-based), schedule, retention; size limit ~10 GB.

**Networking:**
- **Inbound:** access restrictions (IP/service tag rules), **private endpoint**.
- **Outbound:** **VNet integration** lets the app reach resources in a VNet. It does **not** give the app a private inbound address.

**Deployment slots:**
- Standard+ only. Slots are live apps with their own hostnames.
- **Swap** exchanges content and settings with warm-up, so there's no downtime. Swap back to roll back.
- **Slot settings** (sticky) stay with the slot (for example a staging DB connection string).
- **Traffic %** can route a portion of production traffic to a slot (testing in production).

### Common confusion

| Question | Answer |
|---|---|
| Plan vs app? | The **plan** is the compute and bill; apps are what run on it. |
| Scale up vs out? | Up = size/tier; out = instance count. |
| Does the plan region matter? | Yes. Plan is regional; apps share its region. |
| Custom domain vs TLS? | Domain mapping (DNS) is separate from a TLS **binding**. |

> [!WARNING]
> **Exam Trap: features depend on the tier**
>
> Slots and autoscale need **Standard or higher**. If the plan is Basic, "add a slot" means scale up first.

> [!WARNING]
> **Exam Trap: custom domain on Free**
>
> Free-tier apps can't map a custom domain. Scale up first.

> [!WARNING]
> **Exam Trap: swap and settings**
>
> By default app settings swap with the content. Mark environment-specific settings as **slot settings** so they stay.

> [!WARNING]
> **Exam Trap: VNet integration ≠ private inbound**
>
> VNet integration is for **outbound** calls. For private **inbound** access use a private endpoint.

### Remember

- Plan = compute. Tier gates features. Slots + autoscale = Standard+. A record = root, CNAME = subdomain. Backup Basic+.

---

## Scenarios

**1. Deploy a new version with zero downtime and an instant rollback.**
Requirement: no downtime, rollback. Capability: App Service **deployment slot + swap** (Standard+). Swap again to roll back.

**2. A stateless web tier must add VMs when CPU is high.**
Requirement: automatic horizontal scale. Capability: **VMSS (Flexible) with autoscale** behind a load balancer.

**3. A queue-triggered worker container should cost nothing when idle.**
Requirement: event-driven, scale to zero. Capability: **Azure Container Apps** with a queue scale rule. ACI can't autoscale.

**4. Two VMs must survive a datacenter outage in the same region.**
Requirement: zone resilience. Capability: **Availability zones** (not an availability set).

**5. A VM's temp disk and cache must be encrypted, too.**
Requirement: encrypt end to end including temp disk. Capability: **encryption at host**.

## Quick Check

**Q1.** You deploy a template to an RG using **Complete** mode. The RG has a resource not in the template. What happens?

A. Nothing  B. The resource is deleted  C. The deployment fails  D. The resource is tagged

<details><summary>Answer</summary>

**B.** Complete mode removes resources that aren't in the template. Incremental leaves them.

</details>

**Q2.** A VM is shut down from within the guest OS. Are you still billed for compute?

A. No  B. Yes, because it's not deallocated  C. Only for the disk  D. Only for the public IP

<details><summary>Answer</summary>

**B.** Only the **Stopped (deallocated)** state stops compute billing.

</details>

**Q3.** Which service runs containers with scale-to-zero based on queue length?

A. ACI  B. ACR  C. Azure Container Apps  D. Availability set

<details><summary>Answer</summary>

**C.** ACR stores images, ACI doesn't autoscale.

</details>

**Q4.** An App Service plan is on the Basic tier. You need a staging slot. What do you do?

A. Add the slot  B. Scale up to Standard or higher  C. Scale out to 2 instances  D. Enable backup

<details><summary>Answer</summary>

**B.** Slots need Standard+. Scaling out doesn't change the tier.

</details>
