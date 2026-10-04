RDS intro
Absolutely. Below is a **beginner-friendly, practical Amazon RDS guide from zero to connecting RDS MySQL from EC2**. I’ll keep the steps simple and include the important AWS concepts, console settings, commands, troubleshooting, and real-time project flow.

# Amazon RDS – Complete Beginner Guide

## 1. What is Amazon RDS?

**Amazon RDS (Relational Database Service)** is an AWS managed service used to run relational databases without manually managing the database server.

Normally, if you install MySQL on an EC2 instance, you have to manage:

- OS
- MySQL installation
- Patching
- Backups
- Storage
- Monitoring
- High availability
- Database maintenance

With RDS, AWS manages much of this for you.

### Supported database engines

RDS supports:

- MySQL
- PostgreSQL
- MariaDB
- Oracle
- Microsoft SQL Server
- Amazon Aurora

For learning, I recommend starting with **MySQL**.

---

# 2. RDS Architecture

A simple real-time architecture looks like this:

```text
                    Internet
                       |
                       |
                  Internet Gateway
                       |
                 Public Subnet
                       |
                  EC2 Instance
                       |
             Private Network / VPC
                       |
              RDS MySQL Database
              Private Subnet
                       |
                 DB Subnet Group
```

The application normally runs on EC2/ECS/EKS and connects to RDS.

```text
User
  |
  v
Load Balancer
  |
  v
EC2 / Application
  |
  | TCP 3306
  v
RDS MySQL
```

---

# 3. Important RDS Terms

Before creating RDS, understand these terms.

| Term | Meaning |
|---|---|
| RDS | Managed relational database service |
| DB Instance | Your running database |
| DB Engine | MySQL, PostgreSQL, etc. |
| DB Instance Class | CPU/RAM configuration |
| Storage | Database disk |
| Endpoint | DNS address used to connect |
| Port | MySQL normally uses 3306 |
| DB Subnet Group | Subnets where RDS can be placed |
| Security Group | Network firewall |
| Multi-AZ | High availability |
| Read Replica | Used to scale read traffic |
| Snapshot | Point-in-time backup copy |
| Automated Backup | AWS-managed backup |
| Parameter Group | Database configuration |
| Option Group | Engine-specific options |

---

# 4. What is a VPC?

RDS runs inside an **Amazon VPC**.

For example:

```text
VPC
10.0.0.0/16
```

Inside the VPC you can have:

```text
VPC: 10.0.0.0/16
       |
       +---- Public Subnet
       |     10.0.1.0/24
       |
       +---- Private Subnet
             10.0.2.0/24
```

For a production architecture, you generally keep RDS in **private subnets**.

---

# 5. What is a DB Subnet Group?

A **DB subnet group** tells RDS which subnets it can use.

For example:

```text
VPC: 10.0.0.0/16

AZ: ap-south-1a
Private Subnet: 10.0.2.0/24

AZ: ap-south-1b
Private Subnet: 10.0.3.0/24
```

DB Subnet Group:

```text
rds-subnet-group
       |
       +--- 10.0.2.0/24
       |
       +--- 10.0.3.0/24
```

### Why two AZs?

For availability.

If one Availability Zone has a problem, RDS can use another AZ depending on the deployment configuration.

**Beginner rule:**

> Create DB subnet groups using subnets in at least **2 Availability Zones**.

---

# 6. What is a Security Group?

A Security Group works like a virtual firewall.

For MySQL:

```text
MySQL
Port: 3306
```

You should **not normally allow**:

```text
0.0.0.0/0
```

Instead, allow the EC2 application's Security Group.

Example:

```text
EC2 Security Group
       |
       | TCP 3306
       v
RDS Security Group
```

RDS inbound rule:

```text
Type: MySQL/Aurora
Protocol: TCP
Port: 3306
Source: EC2-Security-Group
```

This is much better than allowing the entire internet.

---

# 7. Beginner Lab Architecture

Let's build this:

```text
                    Internet
                       |
                       v
                  EC2 Ubuntu
                Public Subnet
                       |
                       | 3306
                       |
                       v
                RDS MySQL
               Private Subnet
                       |
                DB Subnet Group
```

We'll use:

```text
VPC:              10.0.0.0/16

Public Subnet:    10.0.1.0/24
Private Subnet 1: 10.0.2.0/24
Private Subnet 2: 10.0.3.0/24

EC2:              Ubuntu
RDS:              MySQL
RDS Port:         3306
```

---

# 8. Step 1 – Create VPC

Go to:

**AWS Console → VPC → Your VPCs → Create VPC**

Select:

```text
Resources to create: VPC only
Name: my-vpc
IPv4 CIDR: 10.0.0.0/16
```

Click:

**Create VPC**

---

# 9. Step 2 – Create Public Subnet

Go to:

**VPC → Subnets → Create subnet**

Select:

```text
VPC: my-vpc
Subnet name: public-subnet
Availability Zone: ap-south-1a
IPv4 CIDR: 10.0.1.0/24
```

Create subnet.

---

# 10. Step 3 – Create Private Subnet 1

Create another subnet:

```text
Subnet name: private-subnet-1
AZ: ap-south-1a
CIDR: 10.0.2.0/24
```

---

# 11. Step 4 – Create Private Subnet 2

Create another:

```text
Subnet name: private-subnet-2
AZ: ap-south-1b
CIDR: 10.0.3.0/24
```

Now:

```text
VPC
10.0.0.0/16
 |
 +-- Public
 |   10.0.1.0/24
 |
 +-- Private-1
 |   10.0.2.0/24
 |
 +-- Private-2
     10.0.3.0/24
```

---

# 12. Step 5 – Internet Gateway

Go to:

**VPC → Internet Gateways → Create Internet Gateway**

Name:

```text
my-igw
```

Create it.

Then select:

**Actions → Attach to VPC**

Select:

```text
my-vpc
```

---

# 13. Step 6 – Public Route Table

Go to:

**VPC → Route Tables → Create route table**

Name:

```text
public-rt
```

VPC:

```text
my-vpc
```

Create.

Then:

**Routes → Edit routes → Add route**

```text
Destination: 0.0.0.0/0
Target: Internet Gateway
```

Save.

Then associate:

```text
public-subnet
```

with this route table.

---

# 14. Step 7 – Launch EC2

Go to:

**EC2 → Instances → Launch Instance**

Example:

```text
Name: rds-client
AMI: Ubuntu
Instance type: t3.micro
```

Network:

```text
VPC: my-vpc
Subnet: public-subnet
Auto-assign Public IP: Enable
```

Security Group:

```text
SSH
TCP
22
Source: Your IP
```

Launch instance.

---

# 15. Step 8 – Create RDS Security Group

Go to:

**VPC → Security Groups → Create security group**

Name:

```text
rds-mysql-sg
```

VPC:

```text
my-vpc
```

Inbound rule:

```text
Type: MySQL/Aurora
Protocol: TCP
Port: 3306
Source: EC2 Security Group
```

### Important

Don't use:

```text
0.0.0.0/0
```

for production.

Use:

```text
EC2 Security Group
```

as the source.

---

# 16. Step 9 – Create DB Subnet Group

Go to:

**RDS → Subnet groups**

Click:

**Create DB subnet group**

Enter:

```text
Name:
rds-subnet-group

Description:
MySQL RDS subnet group

VPC:
my-vpc
```

Select:

```text
AZ: ap-south-1a
Subnet: private-subnet-1

AZ: ap-south-1b
Subnet: private-subnet-2
```

Click:

**Create**

Now your RDS subnet group is ready.

---

# 17. Step 10 – Create RDS MySQL

Go to:

**RDS → Databases → Create database**

Select:

```text
Choose a database creation method:
Standard create
```

Engine:

```text
MySQL
```

Choose an available MySQL version appropriate for your lab.

---

# 18. Templates

For learning:

```text
Free tier
```

For production:

```text
Production
```

For your beginner practice, use the lowest-cost configuration that is eligible for your account/region.

---

# 19. RDS Settings

DB identifier:

```text
mydb
```

Master username:

```text
admin
```

Password:

```text
YourStrongPassword
```

**Do not share your actual password.**

---

# 20. DB Instance Class

For a small learning environment, choose a small instance class that your account/region supports.

For example:

```text
db.t3.micro
```

Availability and exact free-tier eligibility can vary by AWS account and current AWS pricing.

---

# 21. Storage

Example:

```text
Storage type:
General Purpose SSD

Allocated storage:
20 GiB
```

You can enable:

```text
Storage autoscaling
```

if appropriate.

For a beginner lab, keep the configuration simple.

---

# 22. Connectivity

This is one of the most important sections.

Select:

```text
VPC:
my-vpc
```

DB subnet group:

```text
rds-subnet-group
```

Public access:

```text
No
```

Security group:

```text
rds-mysql-sg
```

Database port:

```text
3306
```

### Recommended architecture

```text
Internet
   |
   v
EC2
   |
   | 3306
   v
RDS
```

RDS does **not** need to be directly accessible from the internet.

---

# 23. Create Database

Review everything.

Then click:

**Create database**

Wait until the status becomes:

```text
Available
```

---

# 24. Find RDS Endpoint

Open:

**RDS → Databases → mydb**

Look for:

```text
Connectivity & security
```

You'll see something similar to:

```text
Endpoint:
mydb.xxxxxxxxxxxx.ap-south-1.rds.amazonaws.com

Port:
3306
```

The endpoint will be different for your database.

---

# 25. Connect to EC2

SSH into EC2.

For Ubuntu:

```bash
ssh -i mykey.pem ubuntu@<EC2-PUBLIC-IP>
```

For Amazon Linux:

```bash
ssh -i mykey.pem ec2-user@<EC2-PUBLIC-IP>
```

---

# 26. Install MySQL Client

### Ubuntu

```bash
sudo apt update
sudo apt install mysql-client -y
```

Verify:

```bash
mysql --version
```

---

# 27. Test Network Connectivity

Before connecting to MySQL, test port 3306.

```bash
nc -zv <RDS-ENDPOINT> 3306
```

Expected:

```text
Connection to <RDS-ENDPOINT> 3306 port [tcp/mysql] succeeded!
```

If `nc` isn't installed:

```bash
sudo apt install netcat-openbsd -y
```

---

# 28. Connect to RDS

Run:

```bash
mysql -h <RDS-ENDPOINT> -u admin -p
```

Example:

```bash
mysql -h mydb.xxxxxxxxx.ap-south-1.rds.amazonaws.com -u admin -p
```

Enter your RDS password.

If successful:

```text
mysql>
```

Congratulations — your EC2 is connected to RDS.

---

# 29. Check Databases

Inside MySQL:

```sql
SHOW DATABASES;
```

---

# 30. Create Database

```sql
CREATE DATABASE demo;
```

Check:

```sql
SHOW DATABASES;
```

---

# 31. Select Database

```sql
USE demo;
```

---

# 32. Create Table

```sql
CREATE TABLE users (
    id INT,
    name VARCHAR(50)
);
```

Check:

```sql
SHOW TABLES;
```

---

# 33. Insert Data

```sql
INSERT INTO users VALUES (1, 'Anil');
```

Insert another:

```sql
INSERT INTO users VALUES (2, 'Kumar');
```

---

# 34. View Data

```sql
SELECT * FROM users;
```

Output:

```text
+----+-------+
| id | name  |
+----+-------+
|  1 | Anil  |
|  2 | Kumar |
+----+-------+
```

---

# 35. Complete RDS Connection Flow

Remember this flow:

```text
EC2
 |
 | DNS lookup
 v
RDS Endpoint
 |
 | TCP 3306
 v
RDS Security Group
 |
 | Allows EC2 SG
 v
RDS MySQL
 |
 v
Database
```

The important relationship is:

```text
EC2 Security Group
        |
        | allowed as source
        v
RDS Security Group
        |
        | TCP 3306
        v
RDS
```

---

# 36. Why Can't I Connect to RDS?

If you receive:

```text
Can't connect to MySQL server
```

check these in order.

### Check 1 – RDS status

RDS should show:

```text
Available
```

---

### Check 2 – Endpoint

Verify the endpoint:

```bash
nslookup <RDS-ENDPOINT>
```

---

### Check 3 – Port

```bash
nc -zv <RDS-ENDPOINT> 3306
```

---

### Check 4 – Security Group

RDS SG should have:

```text
MySQL/Aurora
TCP
3306
Source: EC2-SG
```

---

### Check 5 – VPC

Check that EC2 and RDS are connected through the same VPC or through appropriate VPC connectivity.

---

### Check 6 – Subnet routing

Verify the subnet route tables and network ACLs if connectivity is still failing.

---

# 37. Important Difference: Public vs Private RDS

## Public RDS

```text
Internet
   |
   v
RDS
```

Potentially reachable from outside the VPC if security/network rules permit it.

Not recommended for production databases.

---

## Private RDS

```text
Internet
   X
   |
   |
EC2
 |
 v
RDS
```

This is the normal production architecture.

---

# 38. RDS Backups

RDS provides automated backup capabilities.

You can configure:

```text
Backup retention period
```

For example:

```text
7 days
```

AWS can maintain automated backups according to the configured retention.

---

# 39. RDS Snapshot

A snapshot is a manual backup you can create.

Go to:

**RDS → Databases → mydb → Actions → Take snapshot**

Example:

```text
Snapshot name:
mydb-before-change
```

Later, you can restore the snapshot into a new DB instance.

---

# 40. Multi-AZ

Multi-AZ is primarily for **high availability**.

Architecture:

```text
              RDS
               |
       +-------+-------+
       |               |
      AZ-A            AZ-B
       |               |
   Primary          Standby
```

If the primary database has an infrastructure failure, AWS can fail over to the standby.

### Important

Multi-AZ is **not the same as Read Replica**.

---

# 41. Read Replica

Read replicas are primarily used to scale read workloads.

Example:

```text
Application
    |
    +---------> Primary RDS
    |
    +---------> Read Replica
```

Writes:

```text
Primary
```

Reads can be distributed to replicas depending on the application's design.

---

# 42. Multi-AZ vs Read Replica

| Feature | Multi-AZ | Read Replica |
|---|---|---|
| Main purpose | High availability | Read scaling |
| Primary use | Failover | Read workloads |
| Standby | Yes | No |
| Read traffic | Generally not for scaling | Yes |
| Disaster/failure recovery | Yes | Can help, but different purpose |

---

# 43. RDS Storage

RDS database storage is managed by AWS.

You generally don't do things such as:

```bash
mkfs
mount
/etc/fstab
```

like you would with an EC2 EBS volume.

That's one of the major differences between EC2-hosted MySQL and RDS.

---

# 44. EC2 MySQL vs RDS MySQL

| EC2 MySQL | RDS MySQL |
|---|---|
| You manage OS | AWS manages infrastructure |
| You install MySQL | AWS provides DB engine |
| You manage patches | AWS handles much maintenance |
| Manual backup possible | Automated backups available |
| You manage storage | AWS manages DB storage |
| More control | Less infrastructure control |
| More administration | Less administration |

---

# 45. RDS Monitoring

Go to:

**RDS → Databases → mydb → Monitoring**

You can monitor metrics such as:

```text
CPUUtilization
DatabaseConnections
FreeStorageSpace
FreeableMemory
ReadIOPS
WriteIOPS
ReadLatency
WriteLatency
```

CloudWatch can be used for monitoring and alarms.

---

# 46. Important RDS Metrics

### CPU

```text
CPUUtilization
```

Shows database CPU usage.

### Connections

```text
DatabaseConnections
```

Shows active database connections.

### Storage

```text
FreeStorageSpace
```

Shows available storage.

### Memory

```text
FreeableMemory
```

Shows memory that can potentially be reclaimed.

---

# 47. RDS Parameter Group

A parameter group contains database engine configuration.

Examples include parameters related to:

```text
character set
logging
timeouts
connection settings
```

You can create a custom parameter group when your application needs specific database settings.

---

# 48. RDS Option Group

Some database engines use option groups for additional engine-specific features.

For basic MySQL learning, you usually don't need to modify this.

---

# 49. RDS Encryption

RDS supports encryption at rest.

Conceptually:

```text
Application
     |
     v
Encrypted connection
     |
     v
RDS
     |
     v
Encrypted storage
```

For production environments, encryption should generally be considered part of the security design.

---

# 50. RDS and Secrets

Don't put database passwords directly inside application code.

Bad:

```text
username = admin
password = MyPassword123
```

Better:

```text
AWS Secrets Manager
        |
        v
Application
        |
        v
RDS
```

This is especially useful in production.

---

# 51. RDS Endpoint

You don't normally connect using an IP address.

You use:

```text
RDS Endpoint
```

Example:

```text
mydb.xxxxxxxxx.ap-south-1.rds.amazonaws.com
```

Your application uses:

```text
hostname
port
username
password
database
```

---

# 52. RDS Port Numbers

Common default ports:

| Database | Default Port |
|---|---:|
| MySQL | 3306 |
| PostgreSQL | 5432 |
| MariaDB | 3306 |
| Oracle | 1521 |
| SQL Server | 1433 |

---

# 53. Real-Time DevOps Architecture

As a DevOps engineer, you may encounter:

```text
                    Internet
                       |
                       v
                Application LB
                       |
                       v
                 EC2 / EKS
                       |
                       v
              Application Pods
                       |
                       | 3306
                       v
                  RDS MySQL
                       |
             +---------+---------+
             |                   |
          Backup              Monitoring
             |                   |
          Snapshot           CloudWatch
```

And:

```text
GitHub
   |
   v
GitHub Actions / Jenkins
   |
   v
Docker Image
   |
   v
ECR
   |
   v
EKS
   |
   v
RDS
```

This is a very common cloud/DevOps pattern.

---

# 54. RDS with Kubernetes

For example:

```text
                    EKS
                     |
          +----------+----------+
          |          |          |
         Pod        Pod        Pod
          \          |          /
           \         |         /
            +--------+--------+
                     |
                     | 3306
                     v
                 RDS MySQL
```

The database does **not** have to run inside Kubernetes.

In many production architectures, the application runs in EKS while the relational database is provided by RDS.

---

# 55. Useful MySQL Commands

Once connected:

```sql
SHOW DATABASES;
```

```sql
USE demo;
```

```sql
SHOW TABLES;
```

```sql
DESCRIBE users;
```

```sql
SELECT * FROM users;
```

```sql
INSERT INTO users VALUES (3, 'Test');
```

```sql
UPDATE users SET name='Anil Kumar' WHERE id=3;
```

```sql
DELETE FROM users WHERE id=3;
```

Exit:

```sql
exit;
```

---

# 56. Important AWS CLI Commands

List RDS instances:

```bash
aws rds describe-db-instances
```

Get DB endpoint:

```bash
aws rds describe-db-instances \
--query 'DBInstances[*].Endpoint.Address'
```

Get DB status:

```bash
aws rds describe-db-instances \
--query 'DBInstances[*].[DBInstanceIdentifier,DBInstanceStatus]'
```

---

# 57. Create RDS Snapshot Using CLI

Example:

```bash
aws rds create-db-snapshot \
--db-instance-identifier mydb \
--db-snapshot-identifier mydb-snapshot
```

Check snapshots:

```bash
aws rds describe-db-snapshots
```

---

# 58. Delete RDS

Be careful with this.

Go to:

**RDS → Databases → mydb → Actions → Delete**

AWS may ask about:

```text
Create final snapshot
```

For a lab, you can choose according to whether you need the database again.

For production, don't delete a database without confirming backup/retention requirements.

---

# 59. Most Important Beginner Troubleshooting

### Error 1

```text
ERROR 2003
Can't connect to MySQL server
```

Check:

```text
RDS status
↓
Endpoint
↓
DNS
↓
Security Group
↓
Port 3306
↓
VPC
↓
Subnet
↓
NACL/route
```

---

### Error 2

```text
Access denied for user
```

Usually check:

```text
Username
Password
Authentication
Database permissions
```

---

### Error 3

```text
Unknown host
```

Check:

```bash
nslookup <RDS-ENDPOINT>
```

---

### Error 4

```text
Connection timed out
```

Usually investigate:

```text
Security Group
Network ACL
Routing
Subnet
VPC connectivity
```

---

# 60. Beginner Practical Lab

I recommend doing this exact lab:

```text
1. Create VPC
        ↓
2. Create Public Subnet
        ↓
3. Create 2 Private Subnets
        ↓
4. Create Internet Gateway
        ↓
5. Create Route Table
        ↓
6. Launch Ubuntu EC2
        ↓
7. Create EC2 Security Group
        ↓
8. Create RDS Security Group
        ↓
9. Create DB Subnet Group
        ↓
10. Create RDS MySQL
        ↓
11. Get RDS Endpoint
        ↓
12. SSH into EC2
        ↓
13. Install MySQL Client
        ↓
14. Test port 3306
        ↓
15. Connect to RDS
        ↓
16. Create Database
        ↓
17. Create Table
        ↓
18. Insert Data
        ↓
19. Query Data
        ↓
20. Create Snapshot
        ↓
21. Check CloudWatch Metrics
```

---

# 61. What You Should Learn for a DevOps Job

Since you're learning AWS/DevOps, focus on these RDS topics:

### Beginner

```text
RDS
MySQL
DB Instance
Endpoint
Port 3306
Security Groups
DB Subnet Groups
Private/Public access
Backups
Snapshots
```

### Intermediate

```text
Multi-AZ
Read Replicas
Parameter Groups
CloudWatch
Performance monitoring
Storage autoscaling
Encryption
Secrets Manager
```

### Advanced

```text
RDS failover
Cross-region replicas
Disaster recovery
Performance tuning
Connection pooling
CloudWatch alarms
AWS Backup
Terraform RDS
RDS with EKS
RDS with CI/CD
Database migration
```

---

# 62. One-Line Interview Explanation

If an interviewer asks:

**"What is Amazon RDS?"**

You can answer:

> **Amazon RDS is a fully managed AWS relational database service that simplifies database provisioning, patching, backups, monitoring, scaling, and high availability for engines such as MySQL and PostgreSQL.**

If they ask:

**"How does EC2 connect to RDS?"**

Answer:

> **The application running on EC2 connects to the RDS endpoint over the database port, typically TCP 3306 for MySQL, and the RDS Security Group allows inbound traffic from the EC2 Security Group.**

### Most important diagram to remember

```text
                    AWS VPC
        ┌───────────────────────────────┐
        │                               │
        │       Public Subnet           │
        │                               │
        │         EC2                   │
        │          │                    │
        │          │ TCP 3306           │
        │          ▼                    │
        │    RDS Security Group         │
        │          │                    │
        │          ▼                    │
        │   Private Subnet              │
        │                               │
        │      RDS MySQL                │
        │          │                    │
        │          ▼                    │
        │       Database                │
        │                               │
        └───────────────────────────────┘
```

**Key point:** For a production-style setup, keep **RDS private**, allow **3306 only from the application's Security Group**, use a **DB Subnet Group spanning at least two AZs**, and use **backups/encryption/monitoring** appropriately.




In AWS RDS, **Single Instance, Multi-AZ/Multiple Instances, and Cluster** can be confusing. Here is the beginner-friendly difference.

## 1. Single Instance

Only **one RDS database instance** is running.

```text
Application
     |
     v
 RDS Instance
    MySQL
```

Example:

```text
RDS MySQL
AZ: ap-south-1a
Instance: 1
```

### If the instance fails?

The database becomes unavailable until the issue is resolved.

### Use for:
- Learning
- Development
- Testing
- Small applications

---

## 2. Multi-AZ / Multiple Instances

There are multiple database instances across Availability Zones.

For traditional RDS Multi-AZ DB instance deployment:

```text
             Application
                  |
                  v
            RDS Endpoint
                  |
                  v
        ┌──────────────────┐
        │ Primary Instance │
        │   AZ: 1a         │
        └────────┬─────────┘
                 │
          Synchronous
           replication
                 │
                 v
        ┌──────────────────┐
        │ Standby Instance │
        │   AZ: 1b         │
        └──────────────────┘
```

The **primary** handles database traffic. The standby is primarily for **high availability/failover**, not normal read scaling.

If the primary fails:

```text
Primary fails
     ↓
AWS performs failover
     ↓
Standby becomes primary
     ↓
Application reconnects
```

### Use for:

- Production
- High availability
- Reduced downtime
- Automatic failover

---

# 3. Cluster

A cluster contains multiple database instances/nodes working together, depending on the database technology.

A common AWS example is **Amazon Aurora**.

```text
                  Application
                       |
                       v
                Cluster Endpoint
                       |
             ┌─────────┴─────────┐
             |                   |
             v                   v
        Writer Instance      Reader Instance
             |                   |
             |                   |
             └───────Storage─────┘
```

For example:

```text
Aurora Cluster
     |
     +---- Writer
     |
     +---- Reader
     |
     +---- Reader
```

The writer handles writes, while reader instances can handle read traffic.

---

# Simple Comparison

| Feature | Single Instance | Multi-AZ | Cluster |
|---|---|---|---|
| Instances | 1 | Usually 2 | Multiple possible |
| AZs | Usually 1 | Multiple AZs | Multiple AZs possible |
| High Availability | Low | High | High |
| Automatic failover | Limited | Yes | Yes, depending on engine/configuration |
| Read scaling | No | Not the primary purpose | Yes, with readers |
| Main purpose | Simple DB | HA | HA + scalability |
| Beginner lab | ✅ | ✅ | Advanced |
| Production | Depends | ✅ | ✅ |

---

# Easy Way to Remember

### Single Instance

```text
1 DB
```

**Purpose:** Simple database.

### Multi-AZ

```text
Primary
   +
Standby
```

**Purpose:** **High availability + failover**

### Cluster

```text
Writer
  +
Readers
  +
Shared/cluster storage
```

**Purpose:** **High availability + scalability**

---

## Important AWS terminology

Don't always use **"multi-instance"** as an AWS RDS configuration name.

You will commonly encounter:

**RDS DB instance deployment options**
- Single-AZ DB instance
- Multi-AZ DB instance deployment
- Multi-AZ DB cluster deployment

And separately:

**Aurora**
- Aurora cluster
- Writer instance
- Reader instances
- Cluster endpoints

So for your DevOps learning, the progression I recommend is:

```text
Single-AZ RDS
      ↓
Multi-AZ RDS
      ↓
Aurora Cluster
      ↓
Read Replicas
      ↓
RDS Monitoring + Backup + DR
```
