# Domain 5: Monitor and Maintain Azure Resources (10–15%)

Official skills: monitor resources with Azure Monitor · implement backup and recovery.

Mental model:

```text
See it        → Metrics (numbers) and Logs (events, queried with KQL)
Be told       → Alert rule → Action group (who/what gets notified or run)
Diagnose net  → Network Watcher
Get data back → Azure Backup   (restore to a point in time)
Stay up       → Site Recovery  (fail over to another region)
```

## Know This

- **Metrics** = numeric, lightweight, near real time, collected automatically for most resources. **Logs** = rich events, stored in a **Log Analytics workspace**, queried with **KQL**.
- **Azure Monitor** is the platform. **Log Analytics** is the log store/query tool inside it.
- **Diagnostic settings** send resource logs/metrics to **Log Analytics, Storage, Event Hubs** (or partners). Resource logs aren't collected until you create one.
- **Alert rule** = scope + condition + **action group**. **Action group** = who to notify / what to run.
- **Alert processing rule** = change what happens to fired alerts (suppress during maintenance, add action groups). It doesn't change whether the alert fires.
- **Azure Monitor Agent (AMA)** + **Data collection rules (DCR)** collect guest OS data from VMs. The old Log Analytics (MMA) agent is retired.
- **Recovery Services vault** (VMs, SQL/SAP in VMs, Files, MARS/DPM, **and Site Recovery**) vs **Backup vault** (newer workloads such as Blobs, Disks, PostgreSQL).
- The Recovery Services vault for **VM backup** must be in the **same region as the VM**. For **Site Recovery** the vault lives in the **target (recovery) region**.
- **Soft delete** for Azure Backup keeps deleted backup data **14 additional days** (enhanced soft delete is configurable).
- **Site Recovery sequence:** enable replication → **test failover** → **failover** → **commit** → **re-protect** → failback.

## Understand This

- When the answer is **Backup** (recover data) versus **Site Recovery** (keep the app running elsewhere).
- Why alert creation is **three separate pieces** (signal, rule, action) and where each lives.
- How to choose **metric alert** vs **log alert** vs **activity log alert**.

---

## 1. Azure Monitor: metrics

### What is it?

Numeric time-series data (CPU %, bytes in, queue length). Think: **"how is it behaving right now?"**

### Know for AZ-104

- **Platform metrics** are collected automatically, stored free for a limited time (about 93 days).
- View in **Metrics Explorer**: pick scope, metric, aggregation (avg, min, max, sum, count), time range, **split by dimension**, **filter**.
- Guest OS metrics (memory, disk inside the VM) need **AMA + DCR** (the host only reports CPU, disk ops, network).
- Metrics can feed **alerts** and **autoscale**.

> [!WARNING]
> **Exam Trap: VM memory**
>
> There's no platform memory metric for an Azure VM by default (the host can't see inside the guest). Install **Azure Monitor Agent** (VM insights) to get memory.

### Remember

- Metrics = automatic, numeric, fast. Guest data needs an agent.

---

## 2. Azure Monitor: logs and Log Analytics

### What is it?

Logs are events and records. A **Log Analytics workspace** stores them for **KQL** queries.

### Know for AZ-104

- Log types: **Activity log** (control plane: who did what to which resource at subscription level; available ~90 days), **Resource logs** (operations inside a resource, **need a diagnostic setting**), **Guest OS/app logs** (via AMA + DCR).
- **Diagnostic settings:** choose categories and destinations (**Log Analytics workspace**, **Storage account** for archive, **Event Hubs** for streaming, partner solutions).
- A VM/resource sends data to a workspace; access to the workspace is controlled with RBAC (Log Analytics Reader/Contributor).

### KQL you should be able to read

```kusto
AzureActivity
| where TimeGenerated > ago(24h)
| where OperationNameValue == "MICROSOFT.COMPUTE/VIRTUALMACHINES/DELETE"
| project TimeGenerated, Caller, ResourceGroup, ActivityStatusValue
| order by TimeGenerated desc
```

```kusto
Perf
| where CounterName == "% Processor Time"
| summarize avg(CounterValue) by Computer, bin(TimeGenerated, 5m)
```

Operators to recognize: `where`, `project`, `summarize … by`, `order by`, `take`, `extend`, `ago()`, `bin()`.

### Common confusion: Metrics vs Logs

| Question | Metrics | Logs |
|---|---|---|
| Data | Numbers over time | Records/events with many fields |
| Latency | Near real time | Slightly delayed (minutes) |
| Collected | Automatically (platform) | Needs diagnostic setting / agent |
| Query | Metrics Explorer | **KQL** |
| Best for | Fast alerting, dashboards | Investigation, correlation |

### Common confusion: Azure Monitor vs Log Analytics

| Question | Azure Monitor | Log Analytics |
|---|---|---|
| What is it? | The whole monitoring platform | The **log store + query** component of it |
| Includes | Metrics, logs, alerts, insights, autoscale | Workspaces and KQL |
| Trap | "Monitor" is the umbrella | A workspace isn't the monitoring service itself |

> [!WARNING]
> **Exam Trap: logs don't appear automatically**
>
> Resource logs (for example storage or Key Vault operations) aren't stored until you create a **diagnostic setting** pointing to a destination.

> [!WARNING]
> **Exam Trap: Activity log vs resource logs**
>
> The **Activity log** says "who created or deleted the VM". Resource logs say "what happened *inside* the resource".

### Remember

- Diagnostic setting = the switch for resource logs. KQL for Log Analytics. Activity log = control plane.

---

## 3. Alerts, action groups and alert processing rules

### What is it?

```text
Signal (metric / log query / activity log event)
        ↓ condition met
Alert rule (scope, condition, severity)
        ↓
Action group (email, SMS, push, voice, webhook, Logic App, Function, Runbook, ITSM)
```

### Know for AZ-104

- **Alert rule types:** **metric** (threshold or dynamic), **log search** (KQL result), **activity log** (a control-plane event, for example VM deleted, service health).
- **Scope:** the resources the rule watches (resource, resource group, subscription).
- **Severity:** 0 (critical) to 4 (verbose).
- **Action group:** reusable. Create once, attach to many rules. Needs a **notification** or an **action**.
- **Alert processing rule:** applies to alerts **after they fire**. Use to **suppress notifications** during planned maintenance, or **add an action group** to everything in a scope. It doesn't stop the alert from being created.
- **Stateful** alerts auto-resolve when the condition clears (metric alerts).

### Common confusion: Alert vs action group vs processing rule

| Question | Alert rule | Action group | Alert processing rule |
|---|---|---|---|
| What is it? | The **condition** to watch | **What happens** (notify/run) | **Modifies** handling of fired alerts |
| Reusable? | Per rule | Yes, shared | Yes, scope-based |
| Trap | Without an action group, nobody is told | Doesn't define the condition | Doesn't prevent the alert from firing |

> [!WARNING]
> **Exam Trap: no action group, no notification**
>
> An alert without an action group fires and shows in the portal, but nobody is notified.

> [!WARNING]
> **Exam Trap: maintenance window**
>
> To avoid emails during maintenance, use an **alert processing rule** (suppress), not deleting the alert rule.

### Remember

- Rule = condition. Action group = response. Processing rule = tweak the response.

---

## 4. Azure Monitor Insights

### Know for AZ-104

Curated monitoring views built on Monitor data:

| Insight | Shows | Needs |
|---|---|---|
| **VM insights** | CPU, memory, disk, network, **Map** (process dependencies) | AMA + DCR (and workspace) |
| **Storage insights** | Capacity, latency, availability, errors across accounts | Platform metrics |
| **Network insights** | Topology and health of network resources | Platform metrics |

### Remember

- VM insights = AMA needed. Storage/network insights use platform data.

---

## 5. Network Watcher and Connection Monitor

### Know for AZ-104

Network Watcher is enabled automatically per region when you create a VNet.

| Tool | Answers |
|---|---|
| **IP flow verify** | Is this packet allowed/denied for a VM, and by **which NSG rule**? |
| **NSG diagnostics** | Same idea for VM/VMSS/App Gateway, can add a rule |
| **Next hop** | Where does traffic to this destination go? (route type, IP, route table) |
| **Effective security rules** | Combined NSG rules applied to a NIC |
| **Connection troubleshoot** | Point-in-time test from a source to a destination |
| **Connection monitor** | **Continuous** connectivity and latency between endpoints (including hybrid) |
| **Packet capture** | Capture traffic on a VM/VMSS |
| **VPN troubleshoot** | Diagnose VPN gateways/connections |
| **Topology** | Visual map of the network |
| **Flow logs** | Log IP traffic; **virtual network flow logs** replace NSG flow logs (NSG flow logs retire **Sept 30, 2027**) |

> [!WARNING]
> **Exam Trap: troubleshoot vs monitor**
>
> *Connection troubleshoot* is a one-time check. *Connection monitor* runs continuously and alerts.

> [!WARNING]
> **Exam Trap: IaaS only**
>
> Network Watcher is for IaaS network health, not PaaS or web analytics.

### Remember

- IP flow verify = NSG. Next hop = routing. Connection monitor = continuous.

---

## 6. Azure Backup

### What is it?

Managed backup that stores recovery points in a **vault**. Use it to **recover data to a point in time**.

### Know for AZ-104

**Vaults**

| | Recovery Services vault | Backup vault |
|---|---|---|
| Protects | **Azure VMs**, Azure Files, SQL/SAP HANA in VMs, on-premises (MARS, DPM/MABS) | Newer workloads: **Blobs, Disks, Azure Database for PostgreSQL**, Kubernetes |
| Also used for | **Azure Site Recovery** | Not Site Recovery |
| Policies | Backup policy | Backup policy |

- **Create the vault** in the **same region** as the resources it protects (for VM backup).
- **Storage redundancy** of the vault: LRS, ZRS, **GRS (default)**. Choose it **before** protecting items. **Cross Region Restore** needs GRS.

**Backup policy** = **schedule** (daily/weekly) + **retention** (daily, weekly, monthly, yearly). For VMs, instant restore snapshots are also kept for a few days for fast restores.

**VM backup flow:** extension takes a **snapshot** → data is transferred to the vault. Recovery point consistency: **application-consistent** (Windows via VSS), file-system consistent (Linux, default), or crash-consistent.

**Restore options (VM):** **create a new VM**, **restore disks**, **replace existing**, **file recovery** (mount recovery point to copy files), **Cross Region Restore** (secondary paired region).

**Protection:** **soft delete** is on by default (14 extra days). Enhanced soft delete lets you customize and make it always-on.

**Monitoring:** built-in **alerts** for backup failures through Azure Monitor and **backup reports** (Log Analytics-based). Configure diagnostic settings to send data to a workspace.

### Remember

- Vault same region as VM. Policy = schedule + retention. Soft delete protects the backup itself.

---

## 7. Azure Site Recovery (ASR)

### What is it?

Disaster recovery: **replicate** a running workload to another region and **fail over** when the primary is down.

### Know for AZ-104

- Azure → Azure: replicate VMs to a **secondary region**. Also supports on-premises (VMware, Hyper-V, physical) to Azure.
- The **Recovery Services vault** for ASR is in the **target region**.
- Replication is **continuous**; crash-consistent recovery points are created frequently (about every 5 minutes). App-consistent points are periodic.
- **Recovery plan:** ordered groups of VMs with optional scripts/manual steps, for orchestrated failover.
- **Sequence:**
  1. **Enable replication** (choose target region, resource group, VNet, cache storage).
  2. **Test failover** into an **isolated network**; no impact on production.
  3. **Failover** (planned or unplanned) to the secondary.
  4. **Commit**.
  5. **Re-protect** (reverse replication) and later **fail back**.

### Common confusion: Azure Backup vs Site Recovery

| Question | Azure Backup | Site Recovery |
|---|---|---|
| Purpose | **Restore data** to a point in time | **Keep workloads running** via failover |
| Protects from | Deletion, corruption, ransomware | Region or site outage |
| Recovery speed | Restore time (minutes–hours) | Minutes (RTO low), data loss ≈ replication lag (RPO low) |
| Copies | Point-in-time backups in vault | Replica VM disks in another region |
| Use together? | Yes. They solve different problems | Yes |
| Trap | Not a failover tool | Not a backup; replicates corruption too |

> [!WARNING]
> **Exam Trap: DR isn't backup**
>
> Replication copies a deleted or encrypted file to the secondary almost immediately. You still need Backup for point-in-time recovery.

> [!WARNING]
> **Exam Trap: vault region**
>
> Backup vault goes with the **source** region; the ASR vault goes in the **target** region.

> [!WARNING]
> **Exam Trap: test failover**
>
> Test failover uses an isolated network and doesn't affect production or replication. Clean up the test.

### Remember

- Backup = recover data. ASR = recover the service. Test failover → failover → commit → re-protect.

---

## Scenarios

**1. Email the on-call team when CPU stays above 85% for 10 minutes on any VM in a resource group.**
Requirement: threshold on a metric + notify. Capability: **metric alert rule** scoped to the RG + **action group** (email/SMS).

**2. Know who deleted a VM last Tuesday.**
Requirement: control-plane history. Capability: **Activity log** (query in Log Analytics if exported).

**3. Stop alert emails for a 2-hour planned maintenance window.**
Requirement: suppress notifications. Capability: **alert processing rule** (suppress) on the scope.

**4. Verify whether an NSG blocks port 3389 from a specific IP to a VM.**
Requirement: packet-level NSG check. Capability: **IP flow verify** (returns the rule).

**5. A VM's disk was corrupted a week ago; recover yesterday's state.**
Requirement: point-in-time restore. Capability: **Azure Backup** (restore disks or create a new VM).

**6. Region outage must not stop the app for long.**
Requirement: regional failover. Capability: **Site Recovery** replicating to a paired region with a **recovery plan**.

## Quick Check

**Q1.** Which Azure Monitor capability lets you run **KQL** queries over collected log data?

A. Metrics Explorer  B. Log Analytics  C. Action group  D. Network Watcher

<details><summary>Answer</summary>

**B.** Metrics Explorer charts numeric data. Action groups define responses.

</details>

**Q2.** An alert rule fires but nobody receives an email. What is the most likely cause?

A. The metric is wrong  B. No action group is attached  C. The VM is deallocated  D. Activity log is disabled

<details><summary>Answer</summary>

**B.** The action group delivers notifications and actions.

</details>

**Q3.** You want to continue operating from another region if the primary region fails. Which service?

A. Azure Backup  B. Azure Site Recovery  C. Soft delete  D. Resource lock

<details><summary>Answer</summary>

**B.** Backup restores data; ASR fails over workloads.

</details>

**Q4.** Where must the Recovery Services vault be to back up an Azure VM?

A. Any region  B. Same region as the VM  C. Paired region  D. In the same VNet

<details><summary>Answer</summary>

**B.** The vault must be in the same region as the VM.

</details>

**Q5.** Which Network Watcher feature **continuously** monitors connectivity and latency between endpoints?

A. IP flow verify  B. Next hop  C. Connection monitor  D. Packet capture

<details><summary>Answer</summary>

**C.** Connection troubleshoot is a single point-in-time test.

</details>
