## Without SSH port also we can connect EC2-Instance
## Without Anisble also we can do task by SSSM in Run Command

Absolutely. If you mean **AWS Systems Manager (SSM)**, here is a practical beginner-to-end setup using an **EC2 instance**.

We will build this example:

> **Use AWS Systems Manager Session Manager to connect to an EC2 instance without SSH or opening port 22.**


## Example architecture

```text
Your Computer
      |
      | AWS Console
      ↓
AWS Systems Manager
      |
      | Session Manager
      ↓
EC2 Instance
      |
      ↓
SSM Agent
      |
      ↓
IAM Role
      |
      ↓
AmazonSSMManagedInstanceCore
```

---

# Step 1 — Create an EC2 instance

Go to:

**AWS Console → EC2 → Instances → Launch instance**

For example:

```text
Name: ssm-demo-server
AMI: Amazon Linux 2023
Instance type: t3.micro
```

For the network, you can use your existing VPC/subnet.

### Security Group

For Session Manager, you **do not need to open SSH port 22**.

You can leave:

```text
Inbound:
No SSH (22) required
```

This is one of the major advantages of Session Manager.

---

# Step 2 — Create an IAM role for EC2

Go to:

**IAM → Roles → Create role**

Select:

```text
Trusted entity type:
AWS service
```

Select:

```text
Use case:
EC2
```

Click **Next**.

Search for:

```text
AmazonSSMManagedInstanceCore
```

Select:

```text
AmazonSSMManagedInstanceCore
```

Click:

**Next → Create role**

Give it a name such as:

```text
EC2-SSM-Role
```

---

# Step 3 — Understand the IAM policy

The important managed policy is:

```text
AmazonSSMManagedInstanceCore
```

It gives the EC2 instance permissions needed to communicate with Systems Manager.

Conceptually:

```text
EC2
 ↓
IAM Role
 ↓
AmazonSSMManagedInstanceCore
 ↓
Systems Manager
```

---

# Step 4 — Attach the IAM role to EC2

Go back to:

**EC2 → Instances**

Select:

```text
ssm-demo-server
```

Then:

**Actions → Security → Modify IAM role**

Select:

```text
EC2-SSM-Role
```

Click:

**Update IAM role**

---

# Reboot the Instance once

# Step 5 — Check SSM Agent

Amazon Linux 2023 AMIs normally include the **SSM Agent**.

You can verify it from the EC2 instance if you have another way to access it:

```bash
sudo systemctl status amazon-ssm-agent
```

You want to see something similar to:

```text
Active: active (running)
```

If necessary:

```bash
sudo systemctl start amazon-ssm-agent
```

And enable it:

```bash
sudo systemctl enable amazon-ssm-agent
```

---

# Step 6 — Make sure the EC2 instance can reach SSM

This is a very important step.

The EC2 instance needs network connectivity to Systems Manager endpoints.

### Option A — Internet/NAT connectivity

If your EC2 instance has appropriate outbound connectivity through an Internet Gateway or NAT Gateway, SSM can communicate with AWS.

You generally need outbound HTTPS:

```text
TCP 443
```

### Option B — Private subnet

If your EC2 instance is in a private subnet without NAT/Internet access, create VPC endpoints for Systems Manager.

Common endpoints are:

```text
com.amazonaws.ap-south-1.ssm
com.amazonaws.ap-south-1.ssmmessages
com.amazonaws.ap-south-1.ec2messages
```

For newer SSM Agent versions, `ssmmessages` is particularly important; AWS's current documentation describes the required endpoint configuration for private connectivity.

---

# Step 7 — Go to Systems Manager

Open:

**AWS Console → Systems Manager**

From the left menu, go to:

**Node Tools → Fleet Manager**

or:

**Node Tools → Managed nodes**

You should eventually see your EC2 instance.

For example:

```text
Instance ID       Ping Status
--------------------------------
i-0123456789abc    Online
```

The important status is:

```text
Online
```

---

# Step 8 — Connect using Session Manager

Go to:

**Systems Manager → Session Manager**

Click:

**Start session**

You should see your EC2 instance.

Select:

```text
ssm-demo-server
```

Click:

**Start session**

A browser-based terminal will open.

You are now connected to your EC2 instance.

---

# Step 9 — Test the connection

Run:

```bash
whoami
```

Then:

```bash
hostname
```

Then:

```bash
uname -a
```

You can also check the OS:

```bash
cat /etc/os-release
```

You should get something similar to:

```text
NAME="Amazon Linux"
VERSION="2023..."
```

---

# Step 10 — Run a real command

For example:

```bash
sudo yum update -y
```

Or create a test file:

```bash
echo "Hello from AWS Systems Manager" | sudo tee /tmp/ssm-test.txt
```

Check it:

```bash
cat /tmp/ssm-test.txt
```

---

# Step 11 — Why SSM is useful

Normally, to connect to an EC2 server using SSH, you might do:

```text
Your computer
     ↓
Internet
     ↓
Port 22
     ↓
EC2
```

With Session Manager:

```text
Your computer
     ↓
AWS Systems Manager
     ↓
SSM Agent
     ↓
EC2
```

So you don't have to expose:

```text
TCP 22
```

to the internet.

---

# Step 12 — SSM Run Command

Session Manager is only one feature of Systems Manager.

Another very useful feature is:

**Run Command**

It lets you execute commands on EC2 instances without opening a terminal session.

Go to:

**Systems Manager → Run Command**

Click:

**Run command**

Choose:

```text
AWS-RunShellScript
```

Under **Targets**, select your EC2 instance.

Under **Commands**, enter:

```bash
hostname
uptime
df -h
```

Click:

**Run**

SSM executes those commands on the EC2 instance.

You'll get output similar to:

```text
hostname
ip-172-31-10-25

uptime
17:20:31 up 2 days, 4:32

df -h
Filesystem      Size  Used Avail Use%
/dev/root        20G  4.2G   16G  22%
```

---

# Step 13 — Example: Install Apache using SSM

This is a good real-world test.

In **Run Command**, select:

```text
AWS-RunShellScript
```

Enter:

```bash
sudo yum install httpd -y
sudo systemctl enable httpd
sudo systemctl start httpd
```

Then verify:

```bash
sudo systemctl status httpd
```

You can do all of this without SSH.

---

# Step 14 — SSM Parameter Store

Another major SSM feature is **Parameter Store**.

For example, suppose your application needs:

```text
DB_HOST
```

Instead of putting it directly into your application code, store it in Parameter Store.

Go to:

**Systems Manager → Parameter Store → Create parameter**

Example:

```text
Name:
 /myapp/db/host

Type:
 String

Value:
 mydatabase.xxxxxx.ap-south-1.rds.amazonaws.com
```

Then your application can retrieve the parameter through AWS APIs/SDKs.

For sensitive values such as passwords, use:

```text
SecureString
```

which can be encrypted using AWS KMS.

---

# Step 15 — SSM Automation

Systems Manager can also automate administrative tasks.

For example:

```text
Automation
    ↓
Stop EC2
    ↓
Create AMI
    ↓
Patch instance
    ↓
Restart instance
```

This is useful when you have many EC2 instances.

---

# The important SSM components

Remember these:

| SSM Feature         | Purpose                        |
| ------------------- | ------------------------------ |
| **Session Manager** | Connect to EC2 without SSH     |
| **Run Command**     | Execute commands remotely      |
| **Parameter Store** | Store configuration/secrets    |
| **Patch Manager**   | Manage OS patches              |
| **Automation**      | Automate operational tasks     |
| **State Manager**   | Maintain desired configuration |
| **Fleet Manager**   | View/manage managed nodes      |

---

# Most important troubleshooting

If your EC2 instance doesn't appear under **Managed nodes**, check these in order:

### 1. IAM Role

EC2 should have:

```text
AmazonSSMManagedInstanceCore
```

attached through its IAM role.

### 2. SSM Agent

Check:

```bash
sudo systemctl status amazon-ssm-agent
```

### 3. Network

Make sure the instance can reach SSM endpoints over:

```text
TCP 443
```

### 4. VPC endpoints

For a private subnet, check:

```text
ssm
ssmmessages
ec2messages
```

where applicable.

### 5. Wait a few minutes

After attaching the role or starting the agent, give SSM a little time to register the instance.

---

## The complete learning path

I recommend learning SSM in this order:

```text
1. EC2
   ↓
2. IAM Role
   ↓
3. AmazonSSMManagedInstanceCore
   ↓
4. SSM Agent
   ↓
5. Managed Node
   ↓
6. Session Manager
   ↓
7. Run Command
   ↓
8. Parameter Store
   ↓
9. Patch Manager
   ↓
10. Automation
```

If you're practicing AWS administration, **Session Manager + Run Command + Parameter Store** are the three SSM features I'd learn first.

