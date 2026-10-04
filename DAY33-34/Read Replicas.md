## Amazon RDS Read Replicas — Beginner Explanation

A **Read Replica** is a copy of your primary database that is used mainly to **handle read requests** and reduce the load on the primary database.

### Simple architecture

```text
                    Application
                        |
             ┌──────────┴──────────┐
             |                     |
          WRITE                   READ
             |                     |
             v                     v
       Primary RDS          Read Replica 1
             |                     |
             | Replication         |
             └────────────────────>|
                                   |
                              Read Replica 2
```

---

## 1. Why do we need Read Replicas?

Suppose your application has:

```text
10,000 requests
```

and most of them are reading data:

```text
9,000 → SELECT
1,000 → INSERT / UPDATE
```

If everything goes to the primary:

```text
              10,000 requests
                    |
                    v
              Primary RDS
                    |
                 HIGH LOAD
```

You can use a Read Replica:

```text
              Application
             /           \
        WRITE             READ
          |                 |
          v                 v
     Primary RDS       Read Replica
```

Now the read workload can be distributed.

---

# 2. How Read Replica works

The primary database receives changes:

```text
INSERT
UPDATE
DELETE
```

Those changes are replicated to the replica.

```text
Primary RDS
    |
    | Replication
    v
Read Replica
```

The replica can then serve read queries.

Example:

```sql
SELECT * FROM users;
```

can be sent to the replica.

---

# 3. Important: Read Replica is NOT the same as Multi-AZ

This is very important for interviews.

### Multi-AZ

```text
Primary
   |
   | synchronous replication
   v
Standby
```

Purpose:

**High Availability / Failover**

---

### Read Replica

```text
Primary
   |
   | replication
   v
Read Replica
```

Purpose:

**Read scalability**

---

## 4. Comparison

| Feature | Multi-AZ | Read Replica |
|---|---|---|
| Main purpose | High availability | Read scaling |
| Read traffic | Not normally used for scaling | Yes |
| Failover | Automatic | Not the primary purpose |
| Replication | Synchronous for traditional Multi-AZ DB instance deployments | Asynchronous |
| Can have multiple | Depending on deployment | Yes, subject to engine/account limits |
| Helps reduce read load | Not primarily | Yes |

---

# 5. Real-Time Example

Suppose you have an e-commerce application.

```text
Users
  |
  v
Application
  |
  +----------------------+
  |                      |
WRITE                    READ
  |                      |
  v                      v
Primary RDS          Read Replica
  |                      |
  |                      |
INSERT                SELECT
UPDATE                SELECT
DELETE                SELECT
```

### Write operations

```sql
INSERT INTO orders VALUES (...);
```

Go to:

```text
Primary RDS
```

### Read operations

```sql
SELECT * FROM products;
```

Can go to:

```text
Read Replica
```

---

# 6. Can we write to a Read Replica?

Normally, **no**.

A read replica is intended for read traffic.

For example:

```sql
SELECT * FROM users;
```

✅ Read

But:

```sql
INSERT INTO users VALUES (...);
```

❌ Not normally allowed on the read replica.

The application should send writes to the primary.

---

# 7. Read Replica Example

Suppose:

```text
Primary:
mydb.xxxxxx.ap-south-1.rds.amazonaws.com
```

Create:

```text
Read Replica:
mydb-replica.xxxxxx.ap-south-1.rds.amazonaws.com
```

Your application could use:

```text
WRITE → Primary Endpoint

READ → Read Replica Endpoint
```

---

# 8. Creating a Read Replica

From the AWS Console:

**RDS → Databases → Select your database**

Then:

**Actions → Create read replica**

You configure things such as:

```text
DB instance identifier:
mydb-read-replica
```

Choose the appropriate:

```text
Region
Availability Zone
Instance class
Storage
Security Group
```

Then create it.

AWS establishes replication from the source database.

---

# 9. Cross-Region Read Replica

Read replicas don't necessarily have to be in the same AWS Region.

Example:

```text
Mumbai Region
ap-south-1
     |
     | replication
     v
Singapore Region
ap-southeast-1
```

This can be useful for:

- Global applications
- Disaster recovery strategies
- Lower-latency reads for users in another region

But cross-region replication has additional considerations such as latency and data transfer costs.

---

# 10. Read Replica with Multiple Replicas

You can have a setup like:

```text
                     Application
                         |
              ┌──────────┴──────────┐
              |                     |
            WRITE                   READ
              |                     |
              v                     v
         Primary RDS         ┌──────────────┐
                             │ Load Balancer │
                             └──────┬───────┘
                                    |
                         ┌──────────┼──────────┐
                         v          v          v
                      Replica1   Replica2   Replica3
```

This is useful when read traffic becomes very large.

---

# 11. Read Replica vs Backup

Don't confuse these.

### Backup

```text
Database
   |
   v
Backup
```

Purpose:

**Recover lost/corrupted data.**

### Read Replica

```text
Primary
   |
   v
Replica
```

Purpose:

**Handle read traffic.**

A Read Replica should **not** be treated as your only backup strategy.

---

# 12. Read Replica vs Snapshot

### Snapshot

Point-in-time backup:

```text
RDS → Snapshot
```

You can use it to restore a database.

### Read Replica

Continuously replicates changes from the source database.

```text
Primary → Replica
```

They solve different problems.

---

# 13. Important Problem: Replication Lag

Because read replicas use replication, there can be a delay.

Example:

```text
10:00:00
Primary:
User = Anil

10:00:01
Application updates:
User = Kumar

Replica:
Still showing Anil
```

After replication catches up:

```text
Replica:
User = Kumar
```

This delay is called **replication lag**.

AWS provides metrics to monitor replica lag, depending on the database engine.

---

# 14. RDS Architecture You Should Remember

For interviews:

```text
                    Application
                         |
             ┌───────────┴───────────┐
             |                       |
           WRITE                    READ
             |                       |
             v                       v
       ┌───────────┐           ┌───────────┐
       │  Primary  │---------->│  Replica  │
       │    RDS    │ Replicate │    RDS    │
       └───────────┘           └───────────┘
```

### Remember these three words:

**Primary → Replication → Read Replica**

---

## One-line interview answer

> **Amazon RDS Read Replicas are read-only database copies that asynchronously replicate data from a source database and are primarily used to scale read workloads and improve application performance.**

And the easiest way to remember the overall RDS concepts is:

```text
Single-AZ
   ↓
Simple database

Multi-AZ
   ↓
High Availability + Failover

Read Replica
   ↓
Read Scalability

Aurora Cluster
   ↓
High Availability + Read Scaling + Cluster Architecture
```
