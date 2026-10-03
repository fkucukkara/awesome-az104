# Domain 1: Manage Azure Identities and Governance (20–25%)

Official skills: Entra users and groups · access to Azure resources (RBAC) · subscriptions and governance.

Mental model for the whole domain:

```text
WHO are you?            → Microsoft Entra ID   (authentication, users, groups)
WHAT can you do?        → Azure RBAC           (authorization on Azure resources)
WHAT is allowed here?   → Azure Policy         (rules on resources)
CAN it be changed?      → Resource lock
WHERE does it sit?      → Management group > Subscription > Resource group > Resource
```

## Know This

- **Entra ID** = identity (users, groups, devices, apps). **RBAC** = permissions on Azure resources.
- RBAC assignment = **security principal + role + scope**. Permissions are **additive** and **inherited downward**.
- Scope order: **management group → subscription → resource group → resource**.
- **Owner** can assign roles. **Contributor** can't. **Reader** is read-only. **User Access Administrator** manages access only.
- **Azure Policy** = governance and compliance. It doesn't grant access.
- **Resource lock** (`CanNotDelete`, `ReadOnly`) applies to **everyone**, including Owners.
- **Tags** are **not inherited** by default.
- **Budgets alert; they don't stop spending.**
- Group-based licensing and dynamic groups need **Entra ID P1** or higher.

## Understand This

- Why RBAC and Policy solve different problems, and when each is the answer.
- Why a lock can "break" things (a ReadOnly lock blocks operations that look like reads).
- Why inheritance means you assign at the **highest scope that makes sense**, and no higher.

---

## 1. Microsoft Entra users and groups

### What is it?

Entra ID is the cloud identity service. It holds users, groups and apps for a tenant. Think: **the directory**.

### Know for AZ-104

- **User types:** member (internal), guest (external, B2B invitation).
- **Create users:** portal, bulk create/invite (CSV), PowerShell/CLI.
- **User properties:** profile data, job info, **usage location** (required before assigning a license).
- **Groups:**
  - Type: **Security** (access, licensing) or **Microsoft 365** (collaboration: mailbox, Teams).
  - Membership: **Assigned**, **Dynamic user**, **Dynamic device** (rules; needs P1+).
- **Licenses:** assign directly to users, or to groups (**group-based licensing**, P1+). Remove a user from the group and the license goes away.
- **External users:** invite by email (B2B). Guests get a limited default permission set. Guests can be added to groups and given RBAC roles.
- **SSPR (self-service password reset):** enabled for **None / Selected group / All** users. Users register authentication methods; admins can reset anyone. Password writeback to on-premises AD needs P1+ and Entra Connect.

### Common confusion

| Question | Answer |
|---|---|
| Can a group's membership be rule-based? | Only dynamic groups (P1+). |
| Which group type for licensing and RBAC? | Security group. |
| Guest vs member? | Guest = external identity invited to your tenant. |

> [!WARNING]
> **Exam Trap: usage location**
>
> A license can't be assigned to a user with no usage location. Group-based licensing errors out for those users.

> [!WARNING]
> **Exam Trap: dynamic groups**
>
> You can't add members manually to a dynamic group. Change the rule instead.

### Remember

- Guest = B2B external user. Dynamic group = rule + P1.
- Licensing needs **usage location**.
- SSPR scope is None / Selected / All.

---

## 2. Azure RBAC: manage access to Azure resources

### What is it?

RBAC answers: **"What can this identity do, and where?"**

```text
Security principal  (user, group, service principal, managed identity)
        +
Role definition     (a set of allowed actions)
        +
Scope               (where it applies)
        =
Role assignment
```

### Know for AZ-104

- **Built-in roles to know:** Owner, Contributor, Reader, User Access Administrator, Virtual Machine Contributor, Storage Blob Data Contributor/Reader, Network Contributor, Monitoring Reader.
- **Inheritance:** assign at a scope and it applies to everything below it.
- **Additive:** effective access is the **union** of all assignments. There's no "deny" in normal role assignments.
- **Interpret assignments:** on **Access control (IAM)**, **Role assignments** shows who has what and whether it's **inherited**; **Check access** shows one identity's effective access.
- **Custom roles** exist when built-in roles don't fit (Actions, NotActions, DataActions, AssignableScopes).
- **Control plane vs data plane:** `Actions` manage the resource; `DataActions` read/write the data inside (blobs, queue messages, secrets).

### Common confusion: Azure RBAC vs Microsoft Entra roles

| Question | Azure RBAC | Entra roles |
|---|---|---|
| What is it? | Permissions on **Azure resources** | Permissions on the **directory** |
| Examples | Owner, Contributor, Reader | Global Administrator, User Administrator |
| Scope | Management group → resource | Tenant (or admin unit) |
| Used for | Create VMs, read storage | Create users, manage licenses, reset passwords |
| AZ-104 trap | Doesn't manage users | Doesn't (by default) grant access to subscriptions |

> [!WARNING]
> **Exam Trap: Contributor can't assign roles**
>
> Contributor manages resources but **cannot** grant access. Owner or User Access Administrator can.

> [!WARNING]
> **Exam Trap: management vs data access**
>
> A management role like Contributor doesn't automatically give data access to blobs through Entra auth. Contributor *can* list storage account keys, and the keys do give data access. Know both sides.

> [!WARNING]
> **Exam Trap: Global Administrator ≠ subscription access**
>
> Global Administrator doesn't automatically have rights on Azure subscriptions. It can **elevate access** to get User Access Administrator at the root scope.

### Remember

- Principal + role + scope. Additive. Inherited.
- Least privilege: **lowest** scope and **narrowest** role that works.

---

## 3. Azure Policy

### What is it?

Policy evaluates resources against rules. It answers **"What is allowed?"**

### Know for AZ-104

- **Policy definition**: the rule (condition + effect). **Initiative**: a group of definitions. **Assignment**: applies a definition or initiative to a scope (with optional exclusions).
- **Effects to know:**

| Effect | Does |
|---|---|
| **Deny** | Blocks a non-compliant create/update |
| **Audit** | Allows it, flags non-compliant |
| **Modify** | Adds/changes properties (for example tags) |
| **DeployIfNotExists** | Deploys a related resource if missing |
| **AuditIfNotExists** | Flags if a related resource is missing |
| **Disabled** | Turns the rule off |

- **Existing resources** are **not** deleted or fixed automatically. They show as non-compliant. **Remediation tasks** fix them for `Modify` and `DeployIfNotExists`.
- **Exemptions** exclude a resource or scope from an assignment.
- Policy evaluates on create/update and on a periodic compliance scan.

### Common confusion: RBAC vs Azure Policy

| Question | RBAC | Azure Policy |
|---|---|---|
| Mental model | "What can this identity do?" | "What is allowed on resources?" |
| Targets | Users, groups, identities | Resources |
| Example | Alice can create VMs | VMs may only be created in West Europe |
| Blocks admin? | Only if they lack the role | Yes, even an Owner can't violate a Deny policy |
| Trap | Doesn't enforce standards | Doesn't grant permissions |

### Common confusion: Resource lock vs Azure Policy

| Question | Lock | Policy |
|---|---|---|
| Purpose | Stop **accidental delete/change** of a specific resource | Enforce **standards** on how resources are configured |
| Works on | Existing resources | Mostly new/updated resources (and audits existing ones) |
| Example | Nobody can delete the prod RG | Every resource must have a `CostCenter` tag |

> [!WARNING]
> **Exam Trap: Deny doesn't clean up**
>
> Assigning a Deny policy doesn't delete existing non-compliant resources. It only blocks new or changed ones.

> [!WARNING]
> **Exam Trap: Policy can't give access**
>
> If the question is "let user X do Y", the answer is RBAC, not Policy.

### Remember

- Policy = compliance/standards. RBAC = permissions. Deny blocks; Audit reports.
- Remediation fixes existing resources.

---

## 4. Resource locks

### What is it?

A lock prevents accidental changes. Think: **"Can this be changed or deleted?"**

### Know for AZ-104

- **CanNotDelete** (shown as *Delete*): can read and modify, can't delete.
- **ReadOnly**: can read, can't modify or delete.
- Apply at **subscription, resource group or resource**. Locks are **inherited** by child resources.
- Apply to **all roles**, including Owner. To remove a lock, you need `Microsoft.Authorization/locks/*` (Owner or User Access Administrator).
- Locks affect the **control plane** (management operations), not data plane operations.

> [!WARNING]
> **Exam Trap: ReadOnly can break things**
>
> A ReadOnly lock on a storage account blocks listing keys (a POST operation), so apps using keys fail. A ReadOnly lock on a VM can block start/stop.

> [!WARNING]
> **Exam Trap: delete the lock first**
>
> To delete a resource group that has a lock (even on a child resource), remove the lock first. An Owner isn't exempt.

### Remember

- Delete lock = no delete. ReadOnly = no change. Inherited. Applies to Owners.

---

## 5. Tags

### What is it?

Name–value pairs for organizing and billing (`Environment=Prod`, `CostCenter=1234`).

### Know for AZ-104

- Applied to resources, resource groups and subscriptions.
- Used in **cost reports** and for filtering.
- **Not inherited** from a resource group or subscription by default.
- To enforce or inherit tags, use **Azure Policy** (`Modify` to inherit a tag, `Deny`/`Audit` to require one).
- Tags are visible to anyone with read access. Never store secrets.

> [!WARNING]
> **Exam Trap: tags don't flow down**
>
> Tagging a resource group does **not** tag its resources. Use Policy to inherit.

### Remember

- Tags = organize + cost. Not inherited. Policy enforces.

---

## 6. Resource groups

### What is it?

A logical container for related resources that share a lifecycle.

### Know for AZ-104

- Each resource is in **exactly one** resource group. Groups **can't be nested**.
- The RG has a **location** (where its metadata is stored). Resources inside can be in **different regions**.
- Deleting an RG deletes **everything inside**.
- **Move resources** between RGs or subscriptions (same tenant). The resource ID changes. Source and destination RGs are locked for writes during the move. Not every resource type can move.
- Use RGs as a scope for RBAC, Policy, locks and cost views.

### Remember

- One RG per resource. No nesting. Delete RG = delete contents. RG region ≠ resource region.

---

## 7. Subscriptions

### What is it?

A billing and access boundary, and the container for resource groups.

### Know for AZ-104

- Each subscription trusts **one** Entra tenant.
- A tenant can have many subscriptions.
- Subscriptions have **quotas/limits** (for example vCPU quotas per region).
- Management tasks: rename, change the management group, transfer billing ownership, cancel, **move resources**.
- **Changing the tenant (directory)** of a subscription **removes RBAC assignments** (they don't carry over).

> [!WARNING]
> **Exam Trap: changing directory**
>
> Moving a subscription to another tenant drops its role assignments and some resources lose their identity links. Plan for it.

### Remember

- Subscription = billing + access boundary. Quotas are per subscription/region. Changing tenant resets RBAC.

---

## 8. Management groups

### What is it?

A container **above** subscriptions to apply RBAC and Policy at scale.

### Know for AZ-104

- Hierarchy: **Tenant root group → management groups → subscriptions → RGs → resources**.
- Up to **6 levels** deep (not counting the root and the subscription level).
- A subscription belongs to **one** management group at a time.
- Policy and RBAC assigned to a management group **inherit** to everything under it.

### Remember

- Management group = governance at scale. Max depth 6. One parent per subscription.

---

## 9. Cost management: alerts, budgets, Advisor

### Know for AZ-104

- **Cost analysis**: see spend by scope, tag, resource.
- **Budgets**: set a spending amount at a scope. Alerts trigger at thresholds (actual or forecasted). **They don't stop or delete resources**; pair with an **action group** to automate.
- **Cost alerts** include budget, credit and anomaly alerts.
- **Azure Advisor** gives recommendations in five categories: **Reliability, Security, Performance, Operational Excellence, Cost** (for example: right-size or shut down underused VMs, buy reservations).

> [!WARNING]
> **Exam Trap: budgets don't enforce**
>
> A budget notifies. It doesn't cap spending. If the question says "prevent" overspend, a budget alone is wrong.

### Remember

- Budget = alert. Advisor = recommendations. Tags + cost analysis = chargeback.

---

## Scenarios

**1. A team must create VMs, but not give anyone else access.**
Requirement: allow VM management at a scope. Capability: RBAC, **Virtual Machine Contributor** at the resource group. Not Contributor (too broad), not Policy (doesn't grant access).

**2. Only resources in West Europe and North Europe may be created across 20 subscriptions.**
Requirement: enforce a standard at scale. Capability: **Azure Policy** (Deny, allowed locations) assigned at a **management group**. RBAC can't restrict regions.

**3. A production RG must not be deleted by mistake, even by admins.**
Requirement: protect from deletion. Capability: **CanNotDelete lock** on the RG. Policy is the wrong tool.

**4. Every resource must carry a `CostCenter` tag, with a default from the RG.**
Requirement: tag enforcement and inheritance. Capability: **Policy** (`Modify` to inherit from RG, plus Deny/Audit for missing). Tags don't inherit on their own.

## Quick Check

**Q1.** Which capability controls **who can perform actions** on Azure resources?

A. Azure Policy  B. Azure RBAC  C. Resource lock  D. NSG

<details><summary>Answer</summary>

**B: Azure RBAC.** Policy controls what resources may look like, locks prevent change/deletion, NSGs filter traffic.

</details>

**Q2.** You apply a `CostCenter` tag to a resource group. Do the resources inside get the tag?

A. Yes, automatically  B. No, unless Policy copies it  C. Only VMs  D. Only if the RG is locked

<details><summary>Answer</summary>

**B.** Tags aren't inherited by default. Use Azure Policy to inherit them.

</details>

**Q3.** A user is Owner of a subscription. A `ReadOnly` lock is on a resource group. Can the user delete a VM in it?

A. Yes, Owner overrides locks  B. No, the lock applies to Owners too  C. Only after a restart  D. Only via PowerShell

<details><summary>Answer</summary>

**B.** Locks apply to all roles. Remove the lock first.

</details>

**Q4.** An admin needs to create users in Entra ID. Which role type applies?

A. Azure RBAC Contributor  B. Azure RBAC Owner  C. Microsoft Entra role (for example User Administrator)  D. Resource lock

<details><summary>Answer</summary>

**C.** Directory tasks use Entra roles. Azure RBAC roles govern Azure resources.

</details>
