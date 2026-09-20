For a beginner, the simplest way to connect **VPC-1 to VPC-2** in AWS is **VPC Peering**.

## Example

We'll create:

```text
VPC-1                         VPC-2
10.0.0.0/16                  20.0.0.0/16
     |                            |
   EC2-1                       EC2-2
10.0.1.10                    20.0.1.10
     \                          /
      \---- VPC Peering -------/
             pcx-xxxx
```

> **Important:** The two VPC CIDR ranges must not overlap.

---

## Step 1: Create VPC-1

Go to:

**AWS Console → VPC → Your VPCs → Create VPC**

Choose:

```text
Name: VPC-1
IPv4 CIDR: 10.0.0.0/16
```

Create VPC.

---

## Step 2: Create VPC-2

Create another VPC:

```text
Name: VPC-2
IPv4 CIDR: 20.0.0.0/16
```

Create VPC.

Now:

```text
VPC-1 → 10.0.0.0/16
VPC-2 → 20.0.0.0/16
```

---

## Step 3: Create VPC Peering

Go to:

**VPC → Peering connections → Create peering connection**

Enter:

```text
Name: VPC1-to-VPC2

Requester VPC:
VPC-1

Accepter VPC:
VPC-2

Region:
This Region
```

Click:

**Create peering connection**

You will get something like:

```text
pcx-0123456789abcdef
```

---

## Step 4: Accept the peering connection

Go to:

**VPC → Peering connections**

Select the new peering connection.

Click:

**Actions → Accept request**

Now status should become:

```text
Active
```

---

# Step 5: Add route in VPC-1

This step is **very important**.

Go to:

**VPC → Route Tables**

Find the route table associated with VPC-1's subnet.

Click:

**Routes → Edit routes → Add route**

Enter:

```text
Destination: 20.0.0.0/16
Target: Peering Connection
        pcx-0123456789abcdef
```

Save.

Your VPC-1 route table becomes:

```text
Destination       Target
10.0.0.0/16       local
20.0.0.0/16       pcx-xxxxxxxx
```

---

# Step 6: Add route in VPC-2

Now go to VPC-2's route table.

Add:

```text
Destination: 10.0.0.0/16
Target: Peering Connection
        pcx-0123456789abcdef
```

Now:

```text
VPC-1 Route Table
10.0.0.0/16 → local
20.0.0.0/16 → PCX
```

and:

```text
VPC-2 Route Table
20.0.0.0/16 → local
10.0.0.0/16 → PCX
```

---

# Step 7: Configure Security Groups

Suppose EC2-1 is:

```text
10.0.1.10
```

and EC2-2 is:

```text
20.0.1.10
```

If you want to SSH from EC2-1 to EC2-2, edit **EC2-2 Security Group**.

Add:

```text
Type: SSH
Protocol: TCP
Port: 22
Source: 10.0.0.0/16
```

Or, more securely, use the **EC2-1 Security Group** as the source if appropriate.

---

# Step 8: Test connectivity

Login to EC2-1:

```bash
ssh ec2-user@10.0.1.10
```

Then test EC2-2:

```bash
ping 20.0.1.10
```

For SSH:

```bash
ssh ec2-user@20.0.1.10
```

You can also test port 22:

```bash
nc -zv 20.0.1.10 22
```

Expected:

```text
Connected to 20.0.1.10:22
```

---

## Complete flow

```text
             VPC Peering
                  |
        ┌─────────┴─────────┐
        │                   │
   VPC-1                 VPC-2
10.0.0.0/16            20.0.0.0/16
        │                   │
      EC2-1               EC2-2
  10.0.1.10             20.0.1.10
        │                   │
        └─────── PCX ───────┘
```

The **3 things you must configure** are:

```text
1. VPC Peering → Active
2. Route tables → Routes pointing to PCX
3. Security Groups → Allow required traffic
```

### Very important beginner point

**VPC Peering alone does NOT create connectivity.**

You need:

```text
Peering
   +
Route table on VPC-1
   +
Route table on VPC-2
   +
Security Group
   =
Connectivity
```

If you want, I can also give you a **complete hands-on lab with 2 VPCs + 2 EC2 instances + exact IP addresses + commands to test the connection**.
