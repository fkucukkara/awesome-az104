# Domain 2: Implement and Manage Storage (15–20%)

Official skills: configure access to storage · configure and manage storage accounts · configure Azure Files and Azure Blob Storage.

Mental model:

```text
Storage account  =  namespace + billing + redundancy + network rules
   ├── Blob       objects in containers   (tiers, lifecycle, versioning)
   ├── Files      SMB/NFS shares          (snapshots, identity-based access)
   ├── Queue / Table (exist, not a focus)
Access = network path (firewall) + identity/credential (RBAC, SAS, key)
```

## Know This

- Storage account name: **3–24 chars, lowercase letters and numbers, globally unique**.
- **LRS** 3 copies, one datacenter · **ZRS** 3 zones, one region · **GRS** LRS + async copy to paired region · **GZRS** ZRS + async copy to paired region. **RA-** = readable secondary.
- **Hot / Cool (30 d) / Cold (90 d) / Archive (180 d, offline)**. Archive can't be the account default and needs rehydration.
- **Archive is not supported on ZRS, GZRS or RA-GZRS** accounts.
- **SAS types:** user delegation (Entra-based, Blob/ADLS), service (key-based, one service), account (key-based, many services).
- **Only a service SAS can use a stored access policy**; revoking = change/delete the policy.
- **Two access keys** exist for rotation. Keys give **full** access.
- **Object replication** needs **versioning** (source and destination) and **change feed** (source).
- Encryption at rest is **always on** (AES-256). You choose **who manages the key**.
- **Storage firewall: IP rules don't apply to same-region Azure traffic**; use virtual network rules or private endpoints.

## Understand This

- Why redundancy (durability) is different from backup and from failover.
- Why a SAS is a delegation of **specific permission for limited time**, and how to revoke it.
- Why network rules and authorization are **two separate gates**: both must allow the request.

---

## 1. Storage accounts

### What is it?

The top-level container for Azure Storage data. It decides **where** data lives, **how** it's copied, and **how** it's reached.

### Know for AZ-104

- **Standard general-purpose v2** is the default choice. **Premium** accounts exist for block blobs, file shares and page blobs (SSD, low latency).
- Choose region, performance (Standard/Premium), redundancy, and the default access tier (Hot/Cool/Cold).
- **Encryption:** Microsoft-managed keys (default) or customer-managed keys in **Key Vault / Managed HSM**. Optional **infrastructure (double) encryption**, which must be chosen **at account creation**. **Encryption scopes** can encrypt at container/blob level.
- **Tools:** **Storage Explorer** (GUI) and **AzCopy** (CLI) for moving data.

```text
azcopy copy "C:\data" "https://<acct>.blob.core.windows.net/<container>?<SAS>" --recursive
azcopy sync  "C:\data" "https://<acct>.blob.core.windows.net/<container>?<SAS>"   # one-way
```

AzCopy authenticates with **Entra ID** (`azcopy login`) or a **SAS**.

### Common confusion

| Question | Answer |
|---|---|
| Can I turn off encryption at rest? | No. It's always on. |
| Can I add double encryption later? | Infrastructure encryption is set at creation. |
| `sync` vs `copy`? | `sync` makes the destination match the source; one direction only. |

> [!WARNING]
> **Exam Trap: account names**
>
> Names are global (they form a DNS name) and can't contain hyphens or uppercase.

### Remember

- GPv2 default. Name rules. Encryption always on; key owner is the choice. AzCopy = SAS or Entra.

---

## 2. Redundancy

### What is it?

How many copies exist and where. Think: **"What can I survive?"**

| Option | Copies | Survives | Secondary readable? |
|---|---|---|---|
| **LRS** | 3 in one datacenter | Drive/rack failure | n/a |
| **ZRS** | 3 across zones in one region | Datacenter (zone) failure | n/a |
| **GRS** | LRS primary + LRS in paired region | Region failure | No (only after failover) |
| **RA-GRS** | Same as GRS | Region failure | **Yes** (read-only) |
| **GZRS** | ZRS primary + LRS in paired region | Zone **and** region failure | No |
| **RA-GZRS** | Same as GZRS | Zone and region failure | **Yes** |

### Know for AZ-104

- Geo-replication to the secondary is **asynchronous**, so data loss is possible on failover (RPO isn't zero).
- The secondary region is the **paired region**, which you don't choose.
- Account **failover** can be initiated by you (or by Microsoft in a disaster).
- Redundancy can be changed after creation (some paths are live migrations, some need a request).
- **Premium** file shares and block blob accounts support only **LRS/ZRS**.

> [!WARNING]
> **Exam Trap: GRS secondary isn't readable**
>
> With plain GRS/GZRS you **can't read from the secondary** until failover. You need RA-GRS or RA-GZRS.

> [!WARNING]
> **Exam Trap: ZRS ≠ region failure**
>
> ZRS protects against a datacenter/zone loss, **not** a regional outage. Pick GZRS for both.

> [!WARNING]
> **Exam Trap: redundancy isn't backup**
>
> Replication copies deletions and corruption, too. Soft delete, versioning and Backup protect against that.

### Remember

- L = one site, Z = zones, G = another region, RA = read the secondary. Async geo copy.

---

## 3. Storage access: network firewall

### What is it?

By default, a storage account is reachable from **all networks** through its public endpoint (if public access is enabled). Firewalls restrict which networks can connect.

### Know for AZ-104

- Set **Public network access** to: Enabled from all networks / **Enabled from selected virtual networks and IP addresses** / Disabled.
- **Virtual network rules:** allow specific subnets. The subnet needs the **Microsoft.Storage service endpoint**.
- **IP rules:** allow **public** IP ranges only (no private ranges).
- **Exceptions:** *Allow trusted Microsoft services* (for example Backup).
- **Private endpoint:** private IP in your VNet for the account. Pair with public access Disabled for fully private access.

> [!WARNING]
> **Exam Trap: IP rules and the same region**
>
> IP network rules have **no effect** on requests from Azure services in the **same region** as the storage account. Use virtual network rules (or a private endpoint) for those.

> [!WARNING]
> **Exam Trap: two gates**
>
> Passing the firewall doesn't grant access. The caller still needs authorization (RBAC, SAS or key). A SAS with a valid signature from a blocked network is still blocked.

### Common confusion: Service endpoint vs private endpoint (for storage)

| Question | Service endpoint | Private endpoint |
|---|---|---|
| What is it? | Subnet setting; traffic uses Azure backbone | A NIC with a **private IP** for the account |
| Target IP | Storage still has its **public** IP | **Private IP** in your VNet |
| Scope | Whole service (for example all Storage in region) | One **specific account** and sub-resource (blob, file…) |
| On-premises access | Doesn't help | Works over VPN/ExpressRoute |
| DNS | Unchanged | Needs private DNS (`privatelink.*`) |

### Remember

- Selected networks + service endpoint for VNets. Private endpoint = private IP. IP rules = public IPs, not same-region.

---

## 4. Storage access: SAS, stored access policies, keys, identities

### What is it?

Ways to **authorize** requests.

| Method | What it is | Notes |
|---|---|---|
| **Entra ID (RBAC)** | Identity-based, data-plane roles | **Recommended**. Storage Blob Data Reader/Contributor/Owner |
| **Shared key / access key** | Account-level secret | Full control, hard to scope |
| **SAS** | Signed URL with limited permission and time | Delegated access |

### Know for AZ-104: SAS

| SAS type | Signed with | Scope | Revoke |
|---|---|---|---|
| **User delegation** | Entra credentials (user delegation key) | Blob / ADLS Gen2 | Revoke the key or the user's role |
| **Service** | Account key | One service (blob, file, queue, table) | **Stored access policy** change/delete, or rotate key |
| **Account** | Account key | Multiple services | **Rotate the key** |

- A SAS defines **permissions, start/expiry, IP range, protocol (HTTPS)**.
- **Stored access policy:** defined on a container, share, queue or table; the SAS references it. Change expiry/permissions or delete it to **revoke** associated SAS tokens. Up to **5** policies per resource.
- **Access keys:** `key1` and `key2` so you can **rotate without downtime** (move apps to key2, regenerate key1).
- The setting **Allow storage account key access** can be disabled to force Entra auth.

> [!WARNING]
> **Exam Trap: revoking an ad hoc SAS**
>
> A SAS **not tied to a stored access policy** can't be revoked individually. The only fix is rotating the account key it was signed with.

> [!WARNING]
> **Exam Trap: Account SAS can't use stored policies**
>
> Only **service SAS** supports stored access policies.

> [!WARNING]
> **Exam Trap: Contributor and keys**
>
> The Contributor role can list account keys. Keys grant full data access, so Contributor effectively has data access.

### Remember

- SAS = delegated, time-limited. Stored policy = revoke handle (service SAS). Rotate keys. Prefer Entra.

---

## 5. Azure Files: identity-based access

### What is it?

Managed file shares over **SMB** (and NFS on Premium). Identity-based access lets users authenticate to SMB shares with directory identities instead of the account key.

### Know for AZ-104

- Identity sources: **on-premises AD DS**, **Microsoft Entra Domain Services**, **Microsoft Entra Kerberos** (hybrid identities).
- **Two layers of permission:**
  1. **Share-level** RBAC: Storage File Data SMB Share Reader / Contributor / Elevated Contributor.
  2. **File/directory-level** NTFS permissions (Windows ACLs).
- SMB uses **TCP 445**. It must be open from the client network.
- Identity-based auth is for **SMB**. NFS shares rely on network controls.

> [!WARNING]
> **Exam Trap: both layers**
>
> Share-level RBAC alone isn't enough to read files if NTFS ACLs don't allow it, and vice versa.

> [!WARNING]
> **Exam Trap: port 445**
>
> A mount that works in Azure but fails from on-premises is often port 445 blocked by the ISP/firewall. Use VPN/ExpressRoute or a private endpoint.

### Remember

- SMB identity = share RBAC + NTFS. Port 445.

---

## 6. Azure Blob Storage

### What is it?

Object storage for unstructured data in **containers**.

### Know for AZ-104

- **Blob types:** block (most data), append (logs), page (VM disks).
- **Container access level:** Private (default), Blob, Container (anonymous read). Anonymous access is disabled at account level by default.
- **Tiers** (block blobs only): see comparison below. Set the **account default** (Hot/Cool/Cold) and override per blob.
- **Soft delete** for blobs and containers: deleted data is recoverable for a retention period.
- **Versioning:** automatically keeps previous versions on overwrite/delete.
- **Snapshots:** manual, read-only point-in-time copy of a blob.
- **Lifecycle management:** rules that **move to cool/cold/archive or delete** based on age (created, modified, or last access when tracking is enabled). Runs on a schedule (about daily).
- **Object replication:** async copy of block blobs between accounts; needs **versioning** on both and **change feed** on the source.

### Hot vs Cool vs Cold vs Archive

| | Hot | Cool | Cold | Archive |
|---|---|---|---|---|
| Use for | Active data | Infrequent | Rare, still online | Long-term, offline |
| Storage cost | Highest | Lower | Lower | Lowest |
| Access cost | Lowest | Higher | Higher | Highest |
| Min retention* | n/a | 30 days | 90 days | 180 days |
| Latency | ms | ms | ms | **Hours** (rehydrate) |
| Can be account default? | Yes | Yes | Yes | **No** |

\*Early-deletion charges apply if removed or moved sooner.

### Versioning vs snapshot vs soft delete

| | What it protects | Created |
|---|---|---|
| **Soft delete** | Accidental delete/overwrite (within retention) | Automatically on delete |
| **Versioning** | Every overwrite and delete, keeps older versions | Automatically |
| **Snapshot** | A chosen point in time | Manually |

> [!WARNING]
> **Exam Trap: archive is offline**
>
> You can't read or modify an archived blob. **Rehydrate** it (change tier, or copy to an online tier) first; this can take up to ~15 hours. Lifecycle policies **cannot** rehydrate.

> [!WARNING]
> **Exam Trap: archive and redundancy**
>
> Archive supports only **LRS, GRS, RA-GRS** accounts, not ZRS/GZRS/RA-GZRS.

> [!WARNING]
> **Exam Trap: object replication prerequisites**
>
> If versioning or change feed isn't enabled, you can't create the replication policy.

> [!WARNING]
> **Exam Trap: lifecycle direction**
>
> Lifecycle rules move data **to cooler** tiers or delete it. They don't bring data back to Hot after access.

### Remember

- Tiers: 30/90/180 days. Archive = offline, rehydrate. Versioning + change feed for replication.

---

## 7. Azure Files management: shares, snapshots, soft delete

### Know for AZ-104

- Create an **Azure Files share** (SMB, or NFS on Premium). Quota and tier (transaction optimized, hot, cool, premium).
- **Share snapshots:** read-only, incremental point-in-time copies of a share. They're stored with the share, so don't use as the only backup.
- **Soft delete for shares:** deleted **shares** are recoverable for a retention period (on by default for new accounts). It protects the **share**, not single-file deletions.
- Azure Backup for Files uses snapshots under the hood.

> [!WARNING]
> **Exam Trap: soft delete scope**
>
> File share soft delete restores a deleted **share**. Recovering one deleted **file** needs a **share snapshot** (or Azure Backup).

### Remember

- Snapshots = file-level recovery. Share soft delete = deleted share recovery.

---

## Scenarios

**1. A VM in a subnet must reach a storage account, and no one else should.**
Requirement: restrict network access. Capability: **firewall "selected networks"** + **service endpoint** (or private endpoint). IP rules don't fit a same-region VM.

**2. A partner needs read access to one container for 48 hours, and you may need to cancel it early.**
Requirement: delegated, time-limited, revocable. Capability: **service SAS tied to a stored access policy** (or user delegation SAS). A bare SAS can't be revoked without rotating a key.

**3. Logs older than 90 days should move to cheaper storage, and be deleted after 3 years.**
Requirement: automatic tier + expiry. Capability: **lifecycle management** rule. Don't script it manually.

**4. Data must survive loss of a datacenter and a region, and be readable from the secondary.**
Requirement: zone + region resilience + read access. Capability: **RA-GZRS**.

**5. Blobs must replicate to a second account in another region for local reads.**
Requirement: copy between accounts. Capability: **object replication** (versioning on both, change feed on source).

## Quick Check

**Q1.** Which redundancy protects against a zone failure **and** a regional outage?

A. ZRS  B. GRS  C. GZRS  D. LRS

<details><summary>Answer</summary>

**C: GZRS.** ZRS covers zones only, GRS covers region only (primary is LRS), LRS covers neither.

</details>

**Q2.** You need to revoke a SAS you issued yesterday without affecting others. What must have been true?

A. It was an account SAS  B. It referenced a stored access policy  C. It used HTTPS  D. It had a short expiry

<details><summary>Answer</summary>

**B.** A service SAS tied to a stored access policy can be revoked by changing or deleting the policy.

</details>

**Q3.** A blob is in the Archive tier. What must you do before reading it?

A. Nothing  B. Rehydrate to Hot/Cool/Cold  C. Enable versioning  D. Switch the account to ZRS

<details><summary>Answer</summary>

**B.** Archive is offline. ZRS accounts can't even use Archive.

</details>

**Q4.** A VM in the same region is blocked by the storage firewall although its public IP is in an IP rule. Why?

A. IP rules need HTTPS  B. IP rules don't apply to same-region Azure traffic  C. The SAS expired  D. The VM needs a public load balancer

<details><summary>Answer</summary>

**B.** Use a virtual network rule (service endpoint) or a private endpoint.

</details>
