# Exam-Day Checklist

No recalled or leaked questions here. This is a method plus a last-hour technical list.

## Before you start

- Use a personal Microsoft account to register (Microsoft recommends this over a work account).
- Take the exam in the language you studied in. If your language isn't available, you can request extra time.
- Check the proctoring rules and your ID in advance.
- Try the [exam sandbox](https://aka.ms/examdemo) once so the interface isn't new.
- Pass mark is **700**; the exam is **100 minutes**.

## For every question

1. **Read the whole scenario** before the options.
2. **Find the actual requirement** (the last sentence usually asks it).
3. **Identify the scope**: subscription, resource group, resource, VNet, region?
4. **Identify the priority:** cheapest, simplest, most secure, least administrative effort, minimum permissions?
5. **Mark absolute words:** *must*, *only*, *cannot*, *minimum*, *at least*, *without*.
6. **Eliminate** options that break an explicit requirement. Then choose among what's left.
7. **Name the layer:** identity? network path? authorization? DNS? region?
8. **Don't add features** the question didn't ask for.

## Watch for

- Two similar services: RBAC/Policy, Backup/ASR, Service/Private endpoint, LB/App Gateway.
- **Direction** of traffic (inbound vs outbound) and **which NSG** (subnet vs NIC).
- **Regional vs global** service. **Same region** requirements (vaults, plans, VM+disk).
- **Control plane vs data plane**.
- **Tier/SKU** requirements (slots, autoscale, Archive, Premium).
- **Defaults** (tags not inherited, NSG default rules, Standard SKU closed by default).
- "**Least privilege**" means the narrowest role at the lowest scope.

## Interactive and multi-part questions

- Read each part. Don't assume previous parts apply.
- In "does the solution meet the goal" sets, judge each proposal against the stated goal only.
- If a case study is shown, note constraints (regions, naming, security) on your scratch area.

## Time

- Don't spend more than about 2 minutes on a first pass. Flag and move on.
- Use remaining time to revisit flagged questions.
- Answer every question. Some sections may not allow going back once submitted; check the on-screen rules in the sandbox.

## Last-hour technical list

**Identity and governance**
- RBAC (what can they do) vs Policy (what's allowed) vs Lock (can't change).
- Contributor can't assign roles. Locks apply to Owners. Tags don't inherit.
- Management group > subscription > RG > resource. Max 6 levels.

**Storage**
- LRS/ZRS/GRS/GZRS/RA-. Tier minimums 30/90/180. Archive needs rehydration.
- SAS revoke = stored access policy (service SAS) or rotate key.
- IP rules ignore same-region traffic. Object replication needs versioning + change feed.

**Compute**
- Complete mode deletes extras. Deallocate to stop billing.
- Set vs zone. ACI vs Container Apps. Slots and autoscale = Standard+.

**Networking**
- Peering non-transitive. NSG: low number wins; subnet then NIC inbound.
- Private endpoint needs DNS. Standard SKU needs NSG. LB probe 168.63.129.16.
- Bastion: `AzureBastionSubnet` /26.

**Monitoring and recovery**
- Alert without action group notifies nobody. Diagnostic settings enable resource logs.
- Backup vs ASR. Vault region rules. Test failover first.

## After the exam

- The result and score report are available per Microsoft's process. If you don't pass, you can retake after 24 hours; see the [retake policy](https://learn.microsoft.com/en-us/credentials/support/retake-policy).
- Renew every 12 months with the free assessment.
