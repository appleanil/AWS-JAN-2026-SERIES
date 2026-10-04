# AWS Transit Gateway — Beginner Level

**AWS Transit Gateway (TGW)** is a service used to **connect multiple VPCs and on-premises networks through one central network hub**.

The easiest way to understand it is:

> **Transit Gateway = Central router for connecting many VPCs.**

---

# 1. Why do we need Transit Gateway?

Suppose you have only two VPCs:

```text
VPC-A  <-------->  VPC-B
```

You can connect them using **VPC Peering**.

But imagine you have:

```text
VPC-1
VPC-2
VPC-3
VPC-4
VPC-5
VPC-6
```

Creating individual peering connections becomes difficult:

```text
VPC1 ---- VPC2
 | \       / |
 |  \     /  |
 |   \   /   |
 VPC3---VPC4
 |  \       |
 VPC5------VPC6
```

This becomes complicated to manage.

With Transit Gateway:

```text
                 VPC-1
                   |
                 VPC-2
                   |
VPC-3 ---- Transit Gateway ---- VPC-4
                   |
                 VPC-5
                   |
                 VPC-6
```

Much simpler.

---

# 2. Transit Gateway Architecture

Think of TGW as a **central router**:

```text
                 VPC-A
                   |
                   |
VPC-B -------- Transit -------- VPC-C
               Gateway
                   |
                   |
                 VPC-D
```

Each VPC connects to the Transit Gateway using a:

**Transit Gateway Attachment**

---

# 3. Important Terms

| Term | Meaning |
|---|---|
| Transit Gateway | Central network hub |
| Attachment | Connection between TGW and VPC/VPN/etc. |
| TGW Route Table | Controls where traffic goes |
| Association | Associates attachment with TGW route table |
| Propagation | Automatically adds routes to TGW route table |
| VPC Peering | Direct connection between two VPCs |
| VPN Attachment | Connects on-premises network through VPN |
| Direct Connect Gateway | Connects AWS TGW with Direct Connect |
| Appliance VPC | Can be used for network/security appliances |

---

# 4. Simple Example

Let's say you have three VPCs.

### VPC 1

```text
VPC-1
CIDR: 10.0.0.0/16
```

### VPC 2

```text
VPC-2
CIDR: 20.0.0.0/16
```

### VPC 3

```text
VPC-3
CIDR: 30.0.0.0/16
```

Create:

```text
Transit Gateway
```

Then:

```text
              Transit Gateway
              /      |       \
             /       |        \
            /        |         \
         VPC-1     VPC-2      VPC-3
       10.0/16    20.0/16    30.0/16
```

Now VPC-1 can communicate with VPC-2 and VPC-3 **if the TGW attachments and route tables are configured to allow it**.

---

# 5. Transit Gateway is Regional

A Transit Gateway is a **Regional service**.

For example:

```text
ap-south-1
     |
 Transit Gateway
```

You can attach multiple VPCs in that Region.

For connecting networks across Regions, AWS provides **Transit Gateway peering** between TGWs.

Example:

```text
Mumbai TGW
10.0.0.0/16
      |
      | TGW Peering
      |
      v
Singapore TGW
20.0.0.0/16
```

---

# 6. Step-by-Step Beginner Lab

Let's create:

```text
VPC-1 → Transit Gateway → VPC-2
```

### Architecture

```text
VPC-1                         VPC-2
10.0.0.0/16                  20.0.0.0/16
    |                            |
    |                            |
    +------ Transit Gateway -----+
```

We'll test communication between EC2 instances.

---

# 7. Step 1 — Create VPC-1

Go to:

**AWS Console → VPC → Your VPCs → Create VPC**

Choose:

```text
Name: VPC-1
CIDR: 10.0.0.0/16
```

Create it.

---

# 8. Step 2 — Create VPC-2

Create another VPC:

```text
Name: VPC-2
CIDR: 20.0.0.0/16
```

Important:

> VPC CIDRs must not overlap.

Good:

```text
VPC-1 = 10.0.0.0/16
VPC-2 = 20.0.0.0/16
```

Bad:

```text
VPC-1 = 10.0.0.0/16
VPC-2 = 10.0.0.0/16
```

---

# 9. Step 3 — Create Transit Gateway

Go to:

**VPC → Transit Gateways**

Click:

**Create Transit Gateway**

Example:

```text
Name:
my-tgw

Description:
Transit Gateway for VPC connectivity
```

For your first lab, you can keep most settings at their defaults unless you have a specific requirement.

Click:

**Create Transit Gateway**

Wait until:

```text
State: Available
```

---

# 10. Step 4 — Create VPC Attachment

Now connect VPC-1 to the TGW.

Go to:

**VPC → Transit Gateway Attachments → Create Transit Gateway Attachment**

Select:

```text
Transit Gateway:
my-tgw

Attachment type:
VPC

VPC:
VPC-1
```

Select the required subnet(s) in the VPC.

Create attachment.

You should eventually see:

```text
VPC-1
   |
   v
TGW Attachment
   |
   v
Transit Gateway
```

---

# 11. Step 5 — Attach VPC-2

Repeat the same process.

```text
VPC-2
   |
   v
TGW Attachment
   |
   v
Transit Gateway
```

Now:

```text
          Transit Gateway
          /             \
         /               \
       VPC-1            VPC-2
```

---

# 12. Very Important — TGW Attachment Is Not Enough

Many beginners make this mistake:

> "I attached both VPCs to Transit Gateway, so they should communicate."

Not necessarily.

You also need **correct routes**.

There are two routing layers to understand.

### VPC route tables

```text
VPC Route Table
      |
      v
Transit Gateway
```

### TGW route table

```text
Transit Gateway
      |
      v
Destination VPC
```

Both need to be configured correctly.

---

# 13. Step 6 — VPC-1 Route Table

Go to:

**VPC → Route Tables**

Find the route table used by the subnet containing your VPC-1 EC2.

Add:

```text
Destination:
20.0.0.0/16

Target:
Transit Gateway → my-tgw
```

Meaning:

```text
10.0.0.0/16
      |
      | Destination 20.0.0.0/16
      v
Transit Gateway
```

---

# 14. Step 7 — VPC-2 Route Table

In VPC-2, add:

```text
Destination:
10.0.0.0/16

Target:
Transit Gateway → my-tgw
```

Now:

```text
VPC-1
10.0.0.0/16
     |
     | 20.0.0.0/16
     v
    TGW
     |
     | 10.0.0.0/16
     v
VPC-2
20.0.0.0/16
```

---

# 15. Step 8 — TGW Route Table

Transit Gateway also has its own route table.

Go to:

**VPC → Transit Gateway Route Tables**

You should have a TGW route table associated with your attachments.

Routes should allow something like:

```text
Destination        Target
10.0.0.0/16        VPC-1 Attachment
20.0.0.0/16        VPC-2 Attachment
```

This tells TGW:

```text
For 10.0.0.0/16 → send to VPC-1

For 20.0.0.0/16 → send to VPC-2
```

---

# 16. Association vs Propagation

These two terms are important.

## Association

An attachment is associated with a TGW route table.

Think:

```text
VPC-1 Attachment
       |
       v
TGW Route Table
```

---

## Propagation

Propagation allows routes from an attachment to be automatically added to the TGW route table.

For example:

```text
VPC-1
10.0.0.0/16
       |
       | Propagation
       v
TGW Route Table
```

The route can appear automatically.

---

# 17. Static Route vs Propagation

### Static

You manually add:

```text
10.0.0.0/16 → VPC-1 attachment
```

### Propagation

The attachment advertises its route to the TGW route table.

For beginner labs, understanding both is important.

---

# 18. Security Groups

Routing alone isn't enough.

Suppose:

```text
EC2-1
10.0.1.10

EC2-2
20.0.1.10
```

You want:

```text
EC2-1 → EC2-2
```

EC2-2's Security Group must allow the required traffic.

For example, for ICMP testing:

```text
Inbound:
Type: All ICMP - IPv4
Source: 10.0.0.0/16
```

Or allow only the specific protocol/port your application needs.

---

# 19. Network ACL

If communication still doesn't work, check:

```text
Security Group
        ↓
Network ACL
        ↓
Route Table
        ↓
TGW Route Table
        ↓
TGW Attachment
```

Network ACLs are stateless, so both inbound and outbound traffic need appropriate rules.

---

# 20. Testing TGW Connectivity

Suppose:

```text
EC2-1:
10.0.1.10

EC2-2:
20.0.1.10
```

From EC2-1:

```bash
ping 20.0.1.10
```

If ICMP is permitted and routing/security rules are correct, you should receive replies.

You can also test an application port:

```bash
nc -zv 20.0.1.10 80
```

or:

```bash
curl http://20.0.1.10
```

depending on what is running on EC2-2.

---

# 21. Transit Gateway vs VPC Peering

This is an important interview question.

### VPC Peering

```text
VPC-A -------- VPC-B
```

For three VPCs:

```text
VPC-A ---- VPC-B
 |           |
 |           |
 +--------- VPC-C
```

More connections are needed as the number of VPCs grows.

### Transit Gateway

```text
             VPC-A
               |
               |
VPC-B ---- TGW ---- VPC-C
               |
               |
             VPC-D
```

Centralized connectivity.

---

# 22. Comparison

| Feature | VPC Peering | Transit Gateway |
|---|---|---|
| Architecture | Point-to-point | Hub-and-spoke |
| Central routing | No | Yes |
| Many VPCs | More complex | Easier |
| Route management | Distributed | Centralized |
| Transitive routing | Not supported through peering | Supported through TGW when routing permits |
| Cost | Lower for simple cases | Additional TGW charges |
| Best for | Small number of connections | Large/multi-VPC networks |

---

# 23. What is Transitive Routing?

This is a very important concept.

Suppose:

```text
VPC-A
  |
  v
VPC-B
  |
  v
VPC-C
```

With VPC Peering, you cannot normally use:

```text
VPC-A → VPC-B → VPC-C
```

as a transit path.

With Transit Gateway:

```text
VPC-A
   |
   v
  TGW
  / \
 v   v
VPC-B VPC-C
```

TGW can provide centralized routing between attached networks when the route tables permit it.

---

# 24. Transit Gateway + On-Premises

TGW isn't only for VPCs.

You can connect:

```text
On-Premises
     |
     | VPN
     v
Transit Gateway
     |
     +---- VPC-1
     |
     +---- VPC-2
     |
     +---- VPC-3
```

For example:

```text
Company Data Center
       |
       | Site-to-Site VPN
       |
       v
Transit Gateway
       |
   +---+---+
   |       |
  VPC1    VPC2
```

This is very common in enterprise environments.

---

# 25. Transit Gateway + Direct Connect

Another architecture:

```text
Corporate Data Center
        |
        |
  AWS Direct Connect
        |
        v
Direct Connect Gateway
        |
        v
Transit Gateway
       / \
      /   \
    VPC1  VPC2
```

This is useful for enterprise hybrid connectivity.

---

# 26. Transit Gateway + Multiple VPCs

A real company might have:

```text
                 Transit Gateway
                       |
       +---------------+---------------+
       |               |               |
       v               v               v
   Dev VPC          Test VPC        Prod VPC
       |               |               |
      EC2             EC2             EKS
                                       |
                                       v
                                      RDS
```

The TGW provides centralized network connectivity, while route tables can control which environments can communicate.

---

# 27. Important Security Concept

Just because two VPCs are attached to the same TGW doesn't mean you should allow unrestricted communication.

For example:

```text
Dev VPC  ----+
             |
Test VPC ----+---- TGW
             |
Prod VPC ----+
```

You can design TGW route tables so that:

```text
Dev → Test     Allowed
Dev → Prod     Blocked
Test → Prod    Allowed
```

depending on your organization's requirements.

This is one reason TGW route tables are important.

---

# 28. Common Beginner Mistakes

### Mistake 1

Using overlapping CIDRs:

```text
VPC-1: 10.0.0.0/16
VPC-2: 10.0.0.0/16
```

❌ Avoid this.

---

### Mistake 2

Creating TGW but not creating attachments.

```text
VPC → TGW
```

The VPC needs a **Transit Gateway Attachment**.

---

### Mistake 3

Creating attachments but forgetting VPC routes.

You need:

```text
VPC route table
       ↓
Transit Gateway
```

---

### Mistake 4

Forgetting TGW route table configuration.

Check:

```text
TGW Route Table
       |
       +-- VPC-1 CIDR
       |
       +-- VPC-2 CIDR
```

---

### Mistake 5

Security Group blocks traffic.

Routing can be correct but SG can still block the connection.

---

# 29. Complete Flow to Remember

For:

```text
EC2 in VPC-1
        ↓
EC2 in VPC-2
```

Traffic flow is:

```text
EC2-1
  ↓
VPC-1 Route Table
  ↓
TGW Attachment
  ↓
Transit Gateway
  ↓
TGW Route Table
  ↓
VPC-2 Attachment
  ↓
VPC-2 Route Table
  ↓
Security Group / NACL
  ↓
EC2-2
```

This is the most important flow to understand.

---

# 30. Interview Answer

### What is AWS Transit Gateway?

> **AWS Transit Gateway is a managed network transit hub that centrally connects multiple VPCs and on-premises networks, simplifying routing and enabling scalable hub-and-spoke network architectures.**

### Why use Transit Gateway?

> **We use Transit Gateway when we need centralized connectivity and routing between many VPCs, VPNs, or other networks instead of creating many individual VPC peering connections.**

### VPC Peering vs Transit Gateway?

> **VPC Peering provides point-to-point connectivity between VPCs, while Transit Gateway provides centralized hub-and-spoke connectivity for multiple VPCs and networks.**

---

# 31. Your Beginner Learning Path

Since you're already practicing **VPC, subnets, route tables, NAT Gateway, security groups, and VPC-to-VPC connectivity**, learn TGW in this order:

```text
VPC
 ↓
Subnet
 ↓
Route Table
 ↓
Internet Gateway
 ↓
NAT Gateway
 ↓
VPC Peering
 ↓
Transit Gateway
 ↓
TGW Attachment
 ↓
TGW Route Table
 ↓
Association
 ↓
Propagation
 ↓
VPN → TGW
 ↓
Direct Connect → TGW
 ↓
Multi-Region TGW Peering
```

### One diagram to memorize

```text
                    AWS Transit Gateway
                           |
        +------------------+------------------+
        |                  |                  |
        v                  v                  v
      VPC-1              VPC-2              VPC-3
   10.0.0.0/16        20.0.0.0/16        30.0.0.0/16
        |                  |                  |
       EC2                EC2                EKS
        |
        +------ centralized routing ------+
```

**Simple definition:**

> **VPC Peering = connect two VPCs.**  
> **Transit Gateway = connect many VPCs/networks through one central hub.**
