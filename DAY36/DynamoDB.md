# AWS DynamoDB – Complete Beginner Guide

If you're learning AWS as a **DevOps Engineer**, DynamoDB is important because it is a **serverless NoSQL database**. Unlike RDS, you don't manage servers, operating systems, or database installations.

---

## 1. What is Amazon DynamoDB?

**Amazon DynamoDB** is a fully managed, serverless **NoSQL database** provided by AWS.

It is designed for applications that need:

- Very fast reads/writes
- Automatic scaling
- High availability
- Large amounts of data
- Low-latency access

Simple architecture:

```text
User
  |
  v
Application
  |
  v
DynamoDB
```

There is no database server for you to install.

---

# 2. RDS vs DynamoDB

This is the first thing you should understand.

| RDS | DynamoDB |
|---|---|
| Relational database | NoSQL database |
| MySQL/PostgreSQL etc. | Key-value/document |
| Tables + rows | Tables + items |
| SQL | DynamoDB API/PartiQL |
| Schema-based | Flexible attributes |
| Manage DB instance configuration | Serverless |
| Usually uses connections | API-based access |
| Joins supported by engines | No traditional joins |
| Good for relational data | Good for high-scale key/value access |

### RDS

```text
Database
   |
 Table
   |
 Rows
   |
 Columns
```

### DynamoDB

```text
Table
  |
  +--- Item
  |      +-- id
  |      +-- name
  |      +-- age
  |
  +--- Item
         +-- id
         +-- name
         +-- city
```

---

# 3. What is NoSQL?

NoSQL means **Not Only SQL**.

Traditional SQL database:

```text
Users
+----+--------+-----+
| ID | Name   | Age |
+----+--------+-----+
| 1  | Anil   | 30  |
| 2  | Kumar  | 28  |
+----+--------+-----+
```

DynamoDB can store items with different attributes.

```text
Item 1:
{
  "id": 1,
  "name": "Anil",
  "age": 30
}
```

Another item could contain:

```text
{
  "id": 2,
  "name": "Kumar",
  "city": "Adoni"
}
```

The attributes don't have to form a traditional fixed relational schema.

---

# 4. Important DynamoDB Terms

You should know these:

| DynamoDB Term | Meaning |
|---|---|
| Table | Container for data |
| Item | One record |
| Attribute | Field/property |
| Partition Key | Main key used to identify/distribute items |
| Sort Key | Optional second key |
| Primary Key | Partition key alone or partition + sort key |
| GSI | Global Secondary Index |
| LSI | Local Secondary Index |
| Capacity | Read/write throughput model |
| TTL | Automatically expire items |
| Streams | Capture item-level changes |
| Point-in-Time Recovery | Continuous backup/recovery |
| On-demand | Pay-per-request capacity mode |

---

# 5. DynamoDB Table

Let's create a simple table:

```text
Table Name:
users
```

Example:

```text
users
```

contains:

```text
ID       Name       City
1        Anil       Adoni
2        Kumar      Hyderabad
3        Ravi       Bangalore
```

But internally, DynamoDB stores these as **items**.

---

# 6. Create Your First DynamoDB Table

Go to:

**AWS Console → DynamoDB**

Then:

**Tables → Create table**

Enter:

```text
Table name:
users
```

Partition key:

```text
id
```

Type:

```text
String
```

For a beginner lab, you can use **On-demand** capacity.

Then click:

**Create table**

---

# 7. What is Partition Key?

This is one of the most important DynamoDB concepts.

Suppose:

```text
Partition Key = id
```

Your items can be:

```text
id = 101
id = 102
id = 103
```

DynamoDB uses the partition key to determine how data is distributed internally.

Think of it as the primary identifier used to locate an item.

---

# 8. Partition Key Must Be Unique

If your table has only:

```text
Partition Key = id
```

then:

```text
id = 101
```

can identify one item.

You cannot have two different items with the same `id` in that table.

Example:

```text
id: 101
name: Anil
```

is valid.

But another item:

```text
id: 101
name: Kumar
```

would overwrite/update the item depending on the operation.

---

# 9. Add an Item

Open:

**DynamoDB → Tables → users → Explore table items**

Click:

**Create item**

Enter:

```text
id = 101
name = Anil
age = 30
city = Adoni
```

Save.

Your item looks conceptually like:

```json
{
  "id": "101",
  "name": "Anil",
  "age": 30,
  "city": "Adoni"
}
```

---

# 10. Add Another Item

Create:

```json
{
  "id": "102",
  "name": "Ravi",
  "age": 28,
  "city": "Hyderabad"
}
```

Now your table contains:

```text
users
 |
 +-- 101 → Anil
 |
 +-- 102 → Ravi
```

---

# 11. Read Data

In the AWS console, you can scan the table to see items.

Conceptually:

```text
Scan users
```

returns multiple items.

---

# 12. Get Item vs Scan

Very important.

### GetItem

You know the key:

```text
id = 101
```

DynamoDB can directly retrieve that item.

```text
GetItem
   |
   v
id = 101
```

This is generally much more efficient than scanning the whole table.

### Scan

```text
Scan
 |
 v
Check many/all items
```

Scan can be expensive for large tables.

### Beginner rule

> **Use key-based queries whenever possible; avoid unnecessary Scan operations on large tables.**

---

# 13. Query

Suppose your table has:

```text
Partition Key = userId
Sort Key = orderId
```

Then:

```text
userId = 101
```

can have:

```text
orderId = 001
orderId = 002
orderId = 003
```

You can query:

```text
userId = 101
```

and retrieve that user's orders.

---

# 14. Sort Key

A Sort Key is an optional second part of the primary key.

Example:

```text
Partition Key: userId
Sort Key: orderId
```

Together:

```text
Primary Key = userId + orderId
```

Example:

```text
userId    orderId
101       001
101       002
101       003
102       001
```

This is called a **composite primary key**.

---

# 15. Why Use a Sort Key?

It lets multiple items share the same partition key.

For example:

```text
Customer 101
   |
   +-- Order 001
   +-- Order 002
   +-- Order 003
```

Very useful for relationships such as:

```text
User → Orders
Customer → Transactions
Device → Sensor readings
```

---

# 16. DynamoDB Data Types

DynamoDB supports several data types.

Common ones:

```text
String
Number
Boolean
Null
List
Map
String Set
Number Set
```

Example:

```json
{
  "id": "101",
  "name": "Anil",
  "age": 30,
  "active": true,
  "skills": [
    "AWS",
    "Docker",
    "Kubernetes"
  ]
}
```

---

# 17. DynamoDB Capacity Modes

There are two important capacity modes.

### On-demand

```text
Requests
   |
   v
DynamoDB
```

AWS automatically handles capacity based on traffic.

Good for:

- Beginners
- Unpredictable workloads
- Development
- Applications with variable traffic

### Provisioned

You specify read/write capacity.

Good when:

- Traffic is predictable
- You want more control over capacity planning

---

# 18. DynamoDB Scaling

DynamoDB is designed for massive scale.

Instead of manually adding database servers:

```text
Server 1
Server 2
Server 3
```

AWS manages the underlying infrastructure.

This is one reason DynamoDB is called a **serverless database**.

---

# 19. DynamoDB High Availability

DynamoDB is designed for high availability and durability across multiple Availability Zones within a Region.

You don't manually create:

```text
Primary DB
Standby DB
```

like you might with a traditional database architecture.

AWS manages the underlying infrastructure.

---

# 20. DynamoDB Global Tables

For applications operating across multiple AWS Regions, DynamoDB supports **Global Tables**.

Example:

```text
              Application
                  |
        +---------+---------+
        |                   |
        v                   v
    Mumbai               Singapore
  DynamoDB              DynamoDB
```

This is useful for global applications requiring multi-Region data access.

---

# 21. DynamoDB TTL

**TTL = Time To Live**

TTL automatically removes expired items after the configured expiration time.

Example:

```text
session
   |
   +-- userId: 101
   +-- expiresAt: timestamp
```

After expiration, DynamoDB can remove the item.

Useful for:

- Sessions
- Temporary data
- Cache-like records
- Expiring tokens
- Temporary events

---

# 22. DynamoDB Streams

DynamoDB Streams capture changes made to items.

Example:

```text
DynamoDB
   |
   | INSERT
   | UPDATE
   | DELETE
   v
DynamoDB Streams
   |
   v
Lambda
```

For example:

```text
User created
     ↓
DynamoDB
     ↓
Stream
     ↓
Lambda
     ↓
Send notification
```

This is very useful in serverless architectures.

---

# 23. DynamoDB + Lambda

A very common AWS architecture:

```text
User
 |
 v
API Gateway
 |
 v
Lambda
 |
 v
DynamoDB
```

For example:

```text
POST /users
     |
     v
API Gateway
     |
     v
Lambda
     |
     v
DynamoDB
```

The Lambda function creates the user item.

---

# 24. DynamoDB + EC2

You can also access DynamoDB from EC2.

```text
EC2
 |
 | AWS SDK / AWS CLI
 v
DynamoDB
```

The EC2 instance should use an **IAM role** with appropriate DynamoDB permissions rather than hardcoding AWS access keys.

---

# 25. Simple AWS CLI Example

Create a table:

```bash
aws dynamodb create-table \
  --table-name users \
  --attribute-definitions AttributeName=id,AttributeType=S \
  --key-schema AttributeName=id,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST
```

Check the table:

```bash
aws dynamodb describe-table \
  --table-name users
```

---

# 26. Put an Item

```bash
aws dynamodb put-item \
  --table-name users \
  --item '{
    "id": {"S": "101"},
    "name": {"S": "Anil"},
    "city": {"S": "Adoni"}
  }'
```

---

# 27. Get an Item

```bash
aws dynamodb get-item \
  --table-name users \
  --key '{"id":{"S":"101"}}'
```

---

# 28. Scan the Table

```bash
aws dynamodb scan \
  --table-name users
```

This returns the items in the table.

Remember:

```text
GetItem → specific key
Query   → key-based retrieval
Scan    → examines items across the table
```

---

# 29. Delete an Item

```bash
aws dynamodb delete-item \
  --table-name users \
  --key '{"id":{"S":"101"}}'
```

---

# 30. Delete the Table

```bash
aws dynamodb delete-table \
  --table-name users
```

Be careful — this deletes the table.

---

# 31. DynamoDB Security

DynamoDB integrates with **IAM**.

Example:

```text
EC2
 |
 | IAM Role
 |
 v
DynamoDB
```

You can give permissions such as:

```text
dynamodb:GetItem
dynamodb:PutItem
dynamodb:UpdateItem
dynamodb:DeleteItem
```

Avoid giving:

```text
dynamodb:*
```

unless there is a specific reason.

Follow the principle of least privilege.

---

# 32. DynamoDB Encryption

DynamoDB supports encryption at rest.

You don't need to manually manage database disks or encryption configuration like you would with a self-managed database server.

---

# 33. Backup and Recovery

DynamoDB supports:

### Point-in-Time Recovery (PITR)

Allows recovery of table data to a point in time within the supported recovery window.

### On-demand backups

You can create backups manually.

---

# 34. DynamoDB vs MongoDB

You may encounter both in DevOps projects.

| DynamoDB | MongoDB |
|---|---|
| AWS managed service | Can be self-managed or managed |
| Serverless option | Typically document database |
| Key-value/document | Document |
| AWS-native | Multi-cloud/open ecosystem |
| IAM integration | MongoDB authentication/authorization |
| Excellent AWS integration | Broad ecosystem |

---

# 35. When Should You Use DynamoDB?

Good use cases:

```text
User profiles
Shopping carts
Sessions
IoT data
Gaming applications
High-scale APIs
Serverless applications
Event-driven applications
```

Example:

```text
10 million users
       |
       v
DynamoDB
```

DynamoDB is designed for very large-scale workloads.

---

# 36. When Should You Use RDS Instead?

If your application needs:

```text
Complex relationships
JOINs
SQL queries
Transactions involving relational data
Existing relational application
```

then RDS may be a better choice.

Example:

```text
Customers
    |
    +--- Orders
          |
          +--- Products
```

A relational database may be more natural here.

---

# 37. DynamoDB Interview Questions

### What is DynamoDB?

> DynamoDB is a fully managed, serverless NoSQL database service from AWS designed for high-performance, low-latency applications at scale.

### What is an item?

> An item is a single record in a DynamoDB table.

### What is an attribute?

> An attribute is a data field within an item.

### What is a partition key?

> A partition key is the primary key component DynamoDB uses to identify and distribute items.

### What is a sort key?

> A sort key is an optional second key used with a partition key to create a composite primary key and organize related items.

### Query vs Scan?

> Query retrieves items based on key conditions, while Scan examines items across a table or index and can be more expensive for large datasets.

### What is TTL?

> TTL automatically expires and removes items after their configured expiration time.

### What are DynamoDB Streams?

> DynamoDB Streams capture item-level changes such as inserts, updates, and deletes, and can trigger services such as Lambda.

---

# 38. Beginner Practical Lab

Do this in order:

```text
1. Open DynamoDB
       ↓
2. Create users table
       ↓
3. Partition key = id
       ↓
4. Select On-demand
       ↓
5. Create table
       ↓
6. Create first item
       ↓
7. Create second item
       ↓
8. Get an item
       ↓
9. Scan table
       ↓
10. Update item
       ↓
11. Delete item
       ↓
12. Create table with Partition + Sort Key
       ↓
13. Practice Query
       ↓
14. Enable TTL
       ↓
15. Learn Streams
       ↓
16. Connect Lambda → DynamoDB
       ↓
17. Learn IAM permissions
```

## The 5 things to remember first

```text
DynamoDB
   |
   +-- Table
   |
   +-- Item
   |
   +-- Attribute
   |
   +-- Partition Key
   |
   +-- Sort Key
```

And remember this comparison:

```text
RDS
 ↓
SQL + relational + joins

DynamoDB
 ↓
NoSQL + key/value/document + massive scale
```

For your AWS/DevOps learning path, after **RDS → Read Replicas → DynamoDB**, the next useful DynamoDB topics are **Partition Key vs Sort Key → Query vs Scan → GSI → TTL → Streams → Lambda integration → IAM → Global Tables**.
