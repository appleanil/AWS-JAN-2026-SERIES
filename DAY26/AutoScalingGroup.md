# AWS Auto Scaling Group Complete Setup with EFS and Load Balancer

## Architecture Overview

```
Customer
   |
   v
Application Load Balancer (ELB)
Security Group : Allow 80 from 0.0.0.0/0
   |
   v
Auto Scaling Group
   |
   v
EC2 Instances
Security Group :
SSH : MyIP
HTTP : From ELB Security Group
   |
   v
Shared Storage
Amazon EFS
Port : 2049 (NFS)
```

Purpose of this architecture

Load Balancer distributes traffic
Auto Scaling automatically manages instances
EFS provides shared storage for all instances

This is a **highly available web architecture**.

---

# Step 1 Create Security Groups

We need **three security groups**.

### 1 ELB Security Group

Inbound Rules

HTTP   80   0.0.0.0/0

Purpose

Allow users from internet to access load balancer.

---

### 2 EC2 Instance Security Group

Inbound Rules

SSH    22    MyIP
HTTP   80    ELB Security Group

Purpose

Allow SSH from admin
Allow web traffic only from Load Balancer.

---

### 3 EFS Security Group

Inbound Rules

NFS   2049   EC2 Security Group

Purpose

Allow EC2 instances to mount EFS.

---

# Step 2 Launch Base EC2 Instance

Launch one EC2 instance.

Configuration

AMI : Amazon Linux
Instance Type : t2.micro
Security Group : EC2 SG
Keypair : your key

Purpose

This instance will act as **base server to configure application**.

---

# Step 3 Install Apache Web Server

Connect to instance

```
ssh -i key.pem ec2-user@public-ip
```

Update system

```
sudo yum update -y
```

Install Apache

```
sudo yum install httpd -y
```

Start service

```
sudo systemctl start httpd
```

Enable on boot

```
sudo systemctl enable httpd
```

Check status

```
sudo systemctl status httpd
```

Test locally

```
curl localhost
```

---

# Step 4 Create Amazon EFS

Go to

AWS Console
EFS
Create File System

Configuration

VPC : same VPC
Subnet : same AZs
Security Group : EFS SG

After creation note

EFS DNS name

Example

```
fs-xxxx.efs.ap-south-1.amazonaws.com
```

---

# Step 5 Install NFS Utilities

EFS works using **NFS protocol**.

Install NFS tools

```
sudo yum install amazon-efs-utils -y
```

or

```
sudo yum install nfs-utils -y
```

---

# Step 6 Mount EFS to Instance

Mount command

```
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport fs-xxxx.efs.ap-south-1.amazonaws.com:/ /var/www/html
```

Verify mount

```
df -h
```

Now Apache document root is using **EFS shared storage**.

---

# Step 7 Permanent Mount

Check mount entry

```
cat /etc/mtab
```

Copy last line.

Edit fstab

```
sudo vi /etc/fstab
```

Add

```
fs-xxxx.efs.ap-south-1.amazonaws.com:/ /var/www/html nfs4 defaults,_netdev 0 0
```

Test

```
sudo mount -a
```

---

# Step 8 Add Website Code

Create sample page

```
sudo vi /var/www/html/index.html
```

Example

```
<h1>Welcome to KK FUNDA DevOps</h1>
<h2>Server IP : $(hostname -I)</h2>
```

Test

```
curl localhost
```

---

# Step 9 Reboot Instance

Test persistence

```
sudo reboot
```

After reboot

Check mount

```
df -h
```

Check website

```
curl localhost
```

---

# Step 10 Create Golden AMI

Stop instance (recommended)

EC2
Actions
Image
Create Image

Name

```
kkfunda-webserver-ami
```

Purpose

AMI will contain

Operating system
Apache configuration
EFS mount
Web content

---

# Step 11 Create Launch Template

Go to

EC2
Launch Templates
Create Launch Template

Configuration

AMI : Golden AMI
Instance Type : t2.micro
Security Group : EC2 SG
Keypair : your key

Important

Do NOT select subnet.

ASG will decide subnet.

---

# Step 12 Create Target Group

Go to

EC2
Target Groups
Create

Configuration

Target type : Instances
Protocol : HTTP
Port : 80
VPC : same VPC

Health check

```
/index.html
```

Do not register instances now.

---

# Step 13 Create Application Load Balancer

Go to

EC2
Load Balancers
Create ALB

Configuration

Type : Internet Facing
Listener : HTTP 80
Security Group : ELB SG
Subnets : Minimum 2 AZ

Listener rule

Forward to Target Group.

---

# Step 14 Create Auto Scaling Group

Go to

EC2
Auto Scaling Groups
Create

Configuration

Launch Template : select template

Network

Select multiple subnets.

Attach Load Balancer

Select Target Group.

Set Capacity

Desired : 2
Minimum : 2
Maximum : 4

Create ASG.

---

# Step 15 Verify Deployment

Go to

Target Groups

Check instance health

Should show

```
Healthy
```

Now access

```
http://ALB-DNS
```

You should see website output.

Refresh page multiple times.

Traffic should rotate between instances.

---

# Step 16 Test Auto Healing

Manually terminate one instance.

EC2
Terminate instance.

ASG will automatically launch new instance.

Check

ASG Activity tab.

New instance should register in Target Group.

---

# Step 17 Manual Scaling

Open Auto Scaling Group

Edit Desired Capacity

Example

2 → 3

ASG launches new instance automatically.

Decrease

3 → 1

ASG terminates instance.

---

# Step 18 Scheduled Scaling

Go to

ASG
Scheduled Actions

Example

Morning 9 AM

Desired Capacity = 4

Night 10 PM

Desired Capacity = 2

This is called

Scheduled Auto Scaling.

---

# Step 19 Dynamic Scaling (Recommended)

Create CloudWatch Alarm.

Example

Metric

CPUUtilization

Condition

CPU > 70% for 5 minutes

Action

Increase instances +1

Another alarm

CPU < 30%

Decrease instances.

## For Dynamically ASG Remove Instances and Add Instances
Policy type: Step scaling
Enabled: Yes
Alarm: Auto-Scaling-Alarm
Condition: CPU > 80%
Action: Remove/Add
Capacity units: 1

---

# Step 20 Final Architecture Benefits

High Availability
Auto Healing
Traffic Distribution
Shared Storage using EFS
Automatic Scaling
Cost Optimization

---

# Real Time Example

Example

Ecommerce website.

During sale

Traffic increases.

Auto Scaling launches more servers.

After sale

Servers terminate automatically.

This saves cost.

