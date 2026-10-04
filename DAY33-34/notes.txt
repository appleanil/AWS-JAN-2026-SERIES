RDS intro
# Amazon RDS – Introduction and Full Setup Guide

This guide covers Amazon RDS concepts, step-by-step instructions to launch a managed database, subnet group configuration, and connecting to the RDS instance from an EC2 server. The explanation is suited for practical training and real-time project implementation.

---

## What is Amazon RDS?

Amazon Relational Database Service (RDS) is a fully managed database service by AWS. It simplifies the setup, operation, and scaling of relational databases in the cloud. RDS supports multiple database engines including:

* MySQL
* PostgreSQL
* MariaDB
* Oracle
* SQL Server
* Amazon Aurora

---

## Key Features of RDS

* Automated backups and snapshots
* Multi-AZ deployments for high availability
* Read replicas for scalability (in some engines)
* Monitoring via CloudWatch
* Integration with IAM, VPC, and Security Groups
* Supports storage autoscaling
* Encryption at rest and in transit

---

## RDS Architecture Overview

* Runs inside a VPC (Virtual Private Cloud)
* Uses **DB Subnet Group** for multi-AZ deployment
* Security Groups define access control
* Endpoints are used to connect from applications or EC2 instances

---

## Step-by-Step: RDS Setup with EC2 Connection

### Step 1: Create a DB Subnet Group

1. Go to **RDS Console → Subnet groups → Create DB subnet group**
2. Name: `rds-subnet-group`
3. Description: `RDS Subnet Group for MySQL`
4. VPC: Select the same VPC where EC2 is running
5. Choose **at least 2 subnets** in different AZs
6. Save the subnet group

> A DB subnet group is mandatory for RDS when using custom VPCs. It tells RDS where it can place instances within subnets.

---

### Step 2: Create RDS Database (MySQL Example)

1. Go to **RDS Console → Databases → Create Database**
2. Choose **Standard Create**
3. Engine type: **MySQL**
4. Version: Use default or latest MySQL version
5. Templates: Choose **Free tier** or **Production**
6. Settings:

   * DB Instance Identifier: `mydb`
   * Master username: `admin`
   * Password: Set a secure password
7. DB Instance Class: `db.t3.micro` (Free tier eligible)
8. Storage: Leave default or enable storage autoscaling
9. Availability & Durability:

   * Enable Multi-AZ for production (optional)
10. Connectivity:

* VPC: Use the same VPC as your EC2
* Subnet Group: Select `rds-subnet-group`
* Public Access: Yes (for testing, No for production)
* VPC Security Group: Select or create one allowing port **3306**

11. Database authentication: Password authentication
12. Monitoring: Enable enhanced monitoring if needed
13. Click **Create Database**

> It takes a few minutes to provision. Note the **RDS Endpoint** from the details page after creation.

---

### Step 3: Connect to RDS from EC2 Instance

> This example uses an Ubuntu EC2 instance. Adjust commands if you're using a different distribution.

1. SSH into your Ubuntu EC2 instance:

```bash
ssh -i mykey.pem ec2-user@<EC2-Public-IP>
```

2. Install MySQL client:

```bash
sudo apt update
sudo apt install mysql-client -y
```

3. Connect to the RDS instance:

```bash
mysql -h <RDS-ENDPOINT> -u admin -p
```

4. Enter the password when prompted

5. Run MySQL commands:

```sql
SHOW DATABASES;
CREATE DATABASE demo;
USE demo;
CREATE TABLE users (id INT, name VARCHAR(50));
INSERT INTO users VALUES (1, 'Test User');
SELECT * FROM users;
```

---

## Security Considerations

* The EC2 instance must be in the same VPC or peered VPC
* Security Group of RDS must allow inbound 3306 from the EC2’s security group
* Don’t expose RDS to public internet in production

---

## Optional Add-ons

* **Automated Backups**: Enable from RDS settings
* **Snapshot Creation**: Use for manual backups or cloning
* **Monitoring**: Enable CloudWatch logs
* **Multi-AZ Failover**: For HA

---

## Summary Table

| Component    | Details                                |
| ------------ | -------------------------------------- |
| Engine       | MySQL (or PostgreSQL, etc.)            |
| Storage      | EBS-based, auto-scaling optional       |
| Availability | Single-AZ or Multi-AZ                  |
| Access       | Via endpoint + username/password       |
| Management   | Fully managed (patching, backup, etc.) |


MYSQL setup

Single instance, mutli instance and cluster

Read replicas
