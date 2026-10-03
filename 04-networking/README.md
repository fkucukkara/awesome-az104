# Domain 4: Implement and Manage Virtual Networking (15–20%)

Official skills: configure and manage virtual networks · configure secure access to virtual networks · configure name resolution and load balancing.

Mental model:

```text
Where things live   → VNet + subnets (+ peering to join VNets)
Where traffic goes  → Routes (system, UDR)
What may pass       → NSG / ASG (filter), Bastion (admin access)
How PaaS is reached → Service endpoint (public IP, subnet identity) or Private endpoint (private IP)
Names               → Azure DNS (public) / Private DNS (inside VNets)
Spreading traffic   → Load Balancer (layer 4)
```

## Know This

- A **VNet is regional**. A **subnet** is a range inside it. Azure reserves **5 IPs per subnet** (first 4 + last).
- **Peering is non-transitive.** Address spaces **must not overlap**.
- **NSG** = stateful allow/deny filter; **lowest priority number wins**; first match stops.
- Inbound: **subnet NSG, then NIC NSG**. Outbound: **NIC NSG, then subnet NSG**.
- Default inbound rules allow VNet and Azure Load Balancer traffic, and **deny the rest**. Default outbound allows VNet and Internet.
- **UDR** next hop types: **Virtual appliance, Virtual network gateway, Virtual network, Internet, None**.
- **Bastion**: browser RDP/SSH over **TLS (443)**, VM needs **no public IP**. Subnet name **`AzureBastionSubnet`**, **/26 or larger** (except Developer SKU).
- **Service endpoint** = traffic stays on Azure backbone, target keeps its public IP. **Private endpoint** = **private IP** in your VNet.
- **Azure DNS** hosts public zones. **Private DNS zone** resolves names inside linked VNets. **Auto-registration** needs a registration link.
- **Standard** Load Balancer and **Standard** public IP are **secure by default** (closed to inbound until an NSG allows). **Basic SKU public IP and Basic Load Balancer were retired on Sept 30, 2025.**
- Health probe source is **168.63.129.16**.

## Understand This

- How an NSG, a UDR and a peering **combine** to decide whether a packet gets from A to B.
- Why a **private endpoint** only works properly when **DNS** returns the private IP.
- Why peering isn't enough for **VNet A ↔ C** through B (no transitivity) and what fixes it.

---

## 1. Virtual networks and subnets

### What is it?

Your private network in Azure. Think: **a regional address space you carve into subnets**.

### Know for AZ-104

- VNet address space (for example `10.0.0.0/16`), subnets within it (`10.0.1.0/24`). Subnets can't overlap.
- **5 reserved addresses per subnet**: network, default gateway, two for Azure DNS, broadcast.
- Resources in the same VNet communicate by default (default NSG rule).
- Some services need **dedicated/delegated subnets** (`AzureBastionSubnet`, `GatewaySubnet`, `AzureFirewallSubnet`).
- **Outbound internet:** the old "default outbound access" is being retired. For VNets created with API versions released after **March 31, 2026**, subnets are **private by default** (the portal already defaults to private). A VM needs an **explicit outbound method**: **NAT gateway** (recommended), public IP on the NIC, Standard LB outbound rules, or a firewall/NVA via UDR. Existing VNets aren't changed.

> [!WARNING]
> **Exam Trap: overlapping address space**
>
> You can't peer VNets with overlapping address spaces. Plan non-overlapping ranges from the start.

> [!WARNING]
> **Exam Trap: "no outbound internet" in a new subnet**
>
> A VM in a private subnet can't reach the internet until you add NAT gateway, public IP, LB outbound rules or a firewall.

### Remember

- VNet regional, subnets non-overlapping, 5 IPs reserved, explicit outbound needed for private subnets.

---

## 2. VNet peering

### What is it?

Connects two VNets so they talk privately over Azure's backbone, as if one network.

### Know for AZ-104

- Works across **regions** (global peering), **subscriptions**, and tenants.
- **Non-transitive:** A↔B and B↔C does **not** give A↔C.
- Needs **non-overlapping** address spaces.
- Peering is configured **in both directions** (the portal can create both).
- Options: **Allow forwarded traffic** (needed for an NVA/firewall in the middle), **Gateway transit / Use remote gateways** (share one VPN gateway).
- Fix for transitivity: peer A↔C directly, or route through a hub **NVA/Azure Firewall** with UDRs, or a hub gateway.

### Common confusion: VNet peering vs VPN Gateway

| Question | VNet peering | VPN Gateway |
|---|---|---|
| Purpose | Connect **Azure VNets** | Connect **on-premises ↔ Azure** (also VNet-to-VNet) |
| Path | Microsoft backbone, private | Encrypted tunnel (IPsec) over internet |
| Latency/bandwidth | Low latency, high bandwidth | Limited by gateway SKU |
| Needs gateway? | No | Yes (`GatewaySubnet`) |
| Trap | Non-transitive | Cost + gateway SKU limits |

> [!WARNING]
> **Exam Trap: transitivity**
>
> Spoke A and spoke C peered to hub B can't talk through B unless B forwards (NVA/firewall + UDR + forwarded traffic).

### Remember

- Peering = private, low-latency, non-transitive, no overlap.

---

## 3. Public and private IP addresses

### Know for AZ-104

- **Private IP** (NIC): from the subnet; **dynamic** (default) or **static** reservation.
- **Public IP:** use **Standard SKU**: static, supports zones, **secure by default**. **Basic SKU public IPs were retired Sept 30, 2025.**
- A Standard public IP **blocks inbound** until an **NSG allows** it.
- A public IP can be attached to a NIC, a load balancer frontend, NAT gateway, Bastion, Application Gateway, etc.

### Common confusion: Public IP vs private IP

| | Public IP | Private IP |
|---|---|---|
| Reachable from | Internet | Inside VNet / connected networks |
| Assigned from | Azure public ranges | Subnet range |
| Cost | Billed | Free |
| Needed for VM? | No (and often shouldn't) | Always |

> [!WARNING]
> **Exam Trap: Standard needs an NSG**
>
> "I added a Standard public IP but can't RDP." Without an NSG rule allowing the port, it's blocked by design.

### Remember

- Standard SKU only. Static. Closed until NSG opens.

---

## 4. User-defined routes (UDR)

### What is it?

Custom routes that override Azure's **system routes**. Think: **"send this traffic via that next hop."**

### Know for AZ-104

- Create a **route table**, add routes (address prefix → next hop), **associate to subnets**.
- **Next hop types:** Virtual appliance (IP), Virtual network gateway, Virtual network, Internet, None (drops traffic).
- **Selection:** **longest prefix match**; on a tie, **UDR > BGP > system**.
- **Forced tunneling / inspection:** `0.0.0.0/0` → Virtual appliance (firewall/NVA).
- An NVA needs **IP forwarding** enabled on its NIC (and in the OS).
- Check with **Effective routes** (NIC) and **Next hop** (Network Watcher).

> [!WARNING]
> **Exam Trap: routes attach to subnets**
>
> A route table does nothing until it's **associated with a subnet**.

> [!WARNING]
> **Exam Trap: next hop None**
>
> `None` blackholes traffic. Use it to block, not to "do nothing".

### Remember

- Longest prefix wins. UDR beats system. NVA needs IP forwarding.

---

## 5. Network security groups and application security groups

### What is it?

An NSG filters traffic by **5-tuple** (source, source port, destination, destination port, protocol).

```text
Source → Destination → Port/Protocol → Priority → Allow/Deny
```

### Know for AZ-104

- Associate an NSG to a **subnet** and/or a **NIC**.
- **Stateful:** if a flow is allowed in one direction, the return traffic is allowed.
- Priority **100–4096**; **lowest number is processed first**; first match wins.
- **Default rules** (can't delete, can override with higher priority):

| Direction | Rule | Priority | Action |
|---|---|---|---|
| Inbound | AllowVnetInBound | 65000 | Allow |
| Inbound | AllowAzureLoadBalancerInBound | 65001 | Allow |
| Inbound | DenyAllInBound | 65500 | Deny |
| Outbound | AllowVnetOutBound | 65000 | Allow |
| Outbound | AllowInternetOutBound | 65001 | Allow |
| Outbound | DenyAllOutBound | 65500 | Deny |

- **Evaluation order:** inbound = **subnet NSG then NIC NSG**; outbound = **NIC NSG then subnet NSG**. Traffic must be allowed by **both**.
- **Service tags** (for example `Storage`, `Internet`, `AzureLoadBalancer`) replace IP lists.
- **ASG:** group NICs (for example *web*, *db*) and use the group as source/destination. Keeps rules readable. All NICs in an ASG must be in the **same VNet**.
- **Effective security rules** (on a NIC) show the combined result.

### Common confusion: NSG vs Azure Firewall

| Question | NSG | Azure Firewall |
|---|---|---|
| What is it? | Basic L3/L4 allow/deny filter | Managed, stateful **network firewall** (L3–L7 features) |
| Attached to | Subnet / NIC | Dedicated `AzureFirewallSubnet` (central) |
| Features | 5-tuple rules, service tags | **FQDN rules**, threat intelligence, NAT, logging, central policy |
| Cost | Free | Paid, per deployment/hour |
| Use | Per-subnet/VM filtering | Central egress/ingress control, hub-and-spoke |

> [!WARNING]
> **Exam Trap: two NSGs, both must allow**
>
> If the subnet NSG allows port 80 but the NIC NSG doesn't, traffic is dropped. Check **effective rules**.

> [!WARNING]
> **Exam Trap: priority direction**
>
> **100 is evaluated before 200.** A broad Deny at priority 100 beats a specific Allow at 200.

> [!WARNING]
> **Exam Trap: VMs in a VNet can talk by default**
>
> The default `AllowVnetInBound` rule lets VMs in the same VNet communicate. To block that, add a **higher-priority Deny** (lower number).

### Remember

- Stateful. Lowest priority number first. Inbound subnet→NIC; outbound NIC→subnet. ASG for tiers.

---

## 6. Azure Bastion

### What is it?

Secure RDP/SSH to VMs **through the portal in your browser** over TLS 443. **No public IP** on the VM and no open RDP/SSH port to the internet.

### Know for AZ-104

- Deploy into a subnet named **`AzureBastionSubnet`**, **/26 or larger** (the Developer SKU doesn't need a dedicated subnet).
- Bastion has its own **Standard public IP**.
- **SKUs:** Developer, Basic, Standard (native client, scaling, shareable links, IP-based connection), Premium (for example session recording, private-only).
- Works with VMs in the **same VNet** or **peered VNets**.
- The VM needs **no** public IP; NSG on the VM subnet must allow RDP/SSH **from the Bastion subnet**.

> [!WARNING]
> **Exam Trap: subnet name and size**
>
> The subnet must be named exactly `AzureBastionSubnet`.

### Remember

- Bastion = no public IP on VMs, `AzureBastionSubnet` /26, TLS 443.

---

## 7. Service endpoints and private endpoints

### What is it?

Two ways for a VNet to reach Azure PaaS services (Storage, SQL…) more securely.

```text
Service endpoint:  Subnet ──(Azure backbone)──► PaaS public endpoint   (service restricted to the subnet)
Private endpoint:  Subnet ──► Private IP (NIC) ──► Specific PaaS resource
```

### Common confusion

| Question | Service endpoint | Private endpoint | Private Link |
|---|---|---|---|
| What is it? | Subnet-level setting extending VNet identity to a service | A **NIC with a private IP** mapped to one resource | The **technology/platform** behind private endpoints |
| Target IP | Service's **public** IP | **Private IP** from your subnet | n/a |
| Granularity | Whole service type in a region | **One resource** and sub-resource | n/a |
| On-premises access | No | **Yes** (VPN/ExpressRoute) | n/a |
| DNS | No change | Needs **private DNS** (`privatelink.*`) | n/a |
| Cost | Free | Paid | n/a |
| Public access | Still exists (firewall restricts) | Can be **disabled** | n/a |

Think: **Private Link is the service, the private endpoint is the thing you create.**

### Know for AZ-104

- **Service endpoint:** enable on the **subnet** for the service (for example `Microsoft.Storage`), then add a **virtual network rule** on the PaaS firewall.
- **Private endpoint:** create for a specific resource and sub-resource (blob, file, sqlServer…). Link it to a **private DNS zone** so the resource FQDN resolves to the private IP.

> [!WARNING]
> **Exam Trap: Private Endpoint ≠ Private Link**
>
> Private Link is the Azure technology. A Private Endpoint is the network interface in your VNet that uses it.

> [!WARNING]
> **Exam Trap: DNS decides everything**
>
> A private endpoint that still resolves to the public IP gives no private access. Use the `privatelink` private DNS zone linked to the VNet.

> [!WARNING]
> **Exam Trap: service endpoints don't help on-premises**
>
> On-premises users can't use a service endpoint. Use a private endpoint or an IP rule.

### Remember

- Service endpoint = public IP, secure path, subnet-based. Private endpoint = private IP, one resource, needs DNS.

---

## 8. Azure DNS

### What is it?

DNS hosting on Azure.

### Know for AZ-104

**Public DNS zone**
- Hosts records for an Internet domain. Azure DNS **doesn't sell domains**; you **delegate** by setting the registrar's **NS records** to the Azure name servers.
- Record types: A, AAAA, CNAME, MX, NS, PTR, SOA, SRV, TXT, CAA.
- A **CNAME can't be at the zone apex** (contoso.com). Use an **alias record** to point apex to an Azure resource.

**Private DNS zone**
- Name resolution **inside VNets** (for example `corp.internal`).
- Link VNets as **registration** links (VM records auto-register) or **resolution-only** links.
- A VNet can be linked to **many** zones for resolution, but to **only one** zone with auto-registration.
- Azure-provided DNS IP is **168.63.129.16**. VNets can also use **custom DNS servers**.

### Common confusion: Azure DNS vs Azure Private DNS

| Question | Azure DNS (public) | Azure Private DNS |
|---|---|---|
| Resolves for | The internet | Linked VNets only |
| Example | `www.contoso.com` | `db.corp.internal` |
| Needs VNet link? | No | **Yes** |
| Delegation at registrar | Yes | No |

> [!WARNING]
> **Exam Trap: apex CNAME**
>
> You can't create a CNAME at the root of a zone. Use an A record or an alias record.

> [!WARNING]
> **Exam Trap: private zone without a VNet link**
>
> A private zone resolves nothing until it's **linked** to a VNet.

### Remember

- Public: delegate NS. Private: link VNet; auto-registration needs a registration link.

---

## 9. Azure Load Balancer

### What is it?

A **layer 4** (TCP/UDP) load balancer. Distributes inbound flows across healthy backend instances.

### Know for AZ-104

- **Public** (internet-facing) vs **Internal** (private frontend IP).
- **Standard SKU**: zone-redundant option, **secure by default**, supports outbound rules, SLA. **Basic SKU was retired on Sept 30, 2025.**
- **Components:** frontend IP · backend pool · **health probe** (TCP/HTTP/HTTPS) · **load-balancing rule** · **inbound NAT rule** · **outbound rule**.
- **Distribution:** default = 5-tuple hash. **Session persistence** (source IP affinity) for stickiness.
- **HA ports:** one rule for all ports on an internal Standard LB.
- A Standard LB blocks inbound until an **NSG allows** it. A public Standard LB needs an **outbound rule** (or NAT gateway) for backend outbound internet.

### Troubleshooting (official skill)

| Symptom | Likely cause |
|---|---|
| Backend marked **unhealthy** | App not listening on the probe port; probe protocol/port mismatch; **NSG/OS firewall blocks 168.63.129.16** |
| Can't reach frontend from internet | Missing **NSG allow** (Standard); no LB rule; wrong frontend; backend unhealthy |
| Backend can't reach internet | No **outbound rule**/NAT gateway |
| A backend VM can't reach its own **internal** LB frontend | **Hairpin limitation**: a VM in the pool can't connect to the LB frontend and be returned to itself |

Use **Load Balancer metrics** (Data path availability, Health probe status) and **Insights**.

### Common confusion: Load Balancer vs Application Gateway vs Front Door

| Question | Load Balancer | Application Gateway | Front Door |
|---|---|---|---|
| Layer | **4** (TCP/UDP) | **7** (HTTP/S) | **7** (HTTP/S) |
| Scope | Regional | Regional | **Global** |
| Features | Fast, any TCP/UDP | URL path routing, multi-site, TLS termination, **WAF** | Global routing, CDN/acceleration, WAF, failover |
| Use | Non-HTTP, VMs | Regional web app routing | Global web apps |
| Trap | Doesn't read URLs | Regional only | Not for non-HTTP |

> [!WARNING]
> **Exam Trap: layer 4 vs layer 7**
>
> If the requirement mentions **URL paths, host headers, SSL termination, WAF**, it's not Load Balancer.

> [!WARNING]
> **Exam Trap: health probe and NSG**
>
> Blocking `168.63.129.16` marks all backends unhealthy. The default `AllowAzureLoadBalancerInBound` rule allows it, so don't override it.

### Remember

- LB = L4, Standard only, probe from 168.63.129.16, NSG needed, outbound rule needed.

---

## 10. Troubleshooting connectivity

Network Watcher tools (details in domain 5):

| Question | Tool |
|---|---|
| Is a packet allowed/denied by NSG? Which rule? | **IP flow verify** |
| Where does traffic go next? | **Next hop** |
| What do rules/routes look like on this NIC? | **Effective security rules / Effective routes** |
| Can A reach B on port X now? | **Connection troubleshoot** (point in time) / **Connection monitor** (over time) |

Troubleshooting order: **DNS → route → NSG → OS firewall → app**.

---

## Scenarios

**1. A VM must reach a storage account privately, and on-premises must too.**
Requirement: private IP, on-premises reachable. Capability: **private endpoint** + private DNS. Service endpoint doesn't work on-premises.

**2. Two spokes need to talk through the hub.**
Requirement: spoke-to-spoke. Capability: **hub NVA/Azure Firewall + UDR (0.0.0.0/0 or spoke prefixes) + forwarded traffic**. Peering alone is non-transitive.

**3. Admins need RDP to VMs with no public IPs and no inbound RDP from internet.**
Requirement: secure admin access. Capability: **Azure Bastion**.

**4. Route HTTP requests `/images/*` and `/api/*` to different pools with WAF.**
Requirement: L7 path routing + WAF. Capability: **Application Gateway**. Load Balancer is L4.

**5. A new VM can't browse the internet.**
Requirement: outbound. Check: private subnet default → add **NAT gateway**; then NSG outbound and UDR (`0.0.0.0/0` → NVA?).

**6. Why can't a VM in subnet-1 reach port 443 on a VM in subnet-2, though subnet-2's NSG allows it?**
Requirement: both gates. Check the **NIC NSG** and the **subnet-1 outbound** NSG; use **IP flow verify**.

## Quick Check

**Q1.** Which provides a **private IP** in your VNet for an Azure Storage account?

A. Service endpoint  B. Private endpoint  C. NSG  D. Route table

<details><summary>Answer</summary>

**B.** A service endpoint keeps the service's public IP.

</details>

**Q2.** VNet A is peered with B, and B is peered with C. Can A reach C?

A. Yes  B. No, peering is not transitive  C. Only with an NSG  D. Only in the same region

<details><summary>Answer</summary>

**B.** Add a direct peering or route via an NVA/gateway.

</details>

**Q3.** An NSG has Allow TCP 443 at priority 200 and Deny all at priority 100. What happens to 443?

A. Allowed  B. Denied  C. Depends on order of creation  D. Allowed if stateful

<details><summary>Answer</summary>

**B.** Lower number = higher precedence.

</details>

**Q4.** Which Azure feature balances HTTP traffic by URL path?

A. Load Balancer  B. Application Gateway  C. NSG  D. Traffic routing in a route table

<details><summary>Answer</summary>

**B.** Load Balancer works at layer 4 and doesn't see URLs.

</details>

**Q5.** Which subnet name is required for Azure Bastion (non-Developer SKU)?

A. `BastionSubnet`  B. `GatewaySubnet`  C. `AzureBastionSubnet`  D. Any name

<details><summary>Answer</summary>

**C.** Minimum /26.

</details>
