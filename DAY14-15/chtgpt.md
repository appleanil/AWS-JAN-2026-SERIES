Sure. Here is the **proper end-to-end workflow** for your scenario:

# EBS Server → Snapshot → New EBS Volume → New EC2 Server

```text
┌─────────────────────────────────────────────────────────┐
│                     EC2 SERVER 1                        │
│                                                         │
│   Root Volume              Additional EBS Volume        │
│   /dev/nvme0n1             /dev/nvme1n1                │
│        │                         │                      │
│        │                         ▼                      │
│        │                    Create XFS                  │
│        │                    Mount /logs                 │
│        │                    Create data                 │
│        │                         │                      │
└────────┼─────────────────────────┼──────────────────────┘
         │                         │
         │                         ▼
         │                  Create Snapshot
         │                         │
         │                         ▼
         │              EBS Snapshot (Backup)
         │                         │
         │                         ▼
         │             Create New EBS Volume
         │                         │
         │                         ▼
┌────────┼─────────────────────────┼──────────────────────┐
│        │                  EC2 SERVER 2                  │
│        │                         │                      │
│   Root Volume              New EBS Volume               │
│   /dev/nvme0n1             /dev/nvme2n1                │
│                                  │                      │
│                                  ▼                      │
│                    Mount with -o nouuid                 │
│                                  │                      │
│                                  ▼                      │
│                        Old Data Available               │
└─────────────────────────────────────────────────────────┘
```

---

# PART 1: Create EC2 Server 1

Launch your first EC2 instance.

After login:

```bash
lsblk
```

Example:

```text
nvme0n1
├─nvme0n1p1    /
├─nvme0n1p127
└─nvme0n1p128
```

This is your **root volume**.

⚠️ Do not format or delete the root volume.

---

# PART 2: Create an Additional EBS Volume

Go to:

```text
AWS Console
    ↓
EC2
    ↓
Elastic Block Store
    ↓
Volumes
    ↓
Create Volume
```

Important: Select the **same Availability Zone** as EC2 Server 1.

For example:

```text
EC2 Server 1 → ap-south-1a
EBS Volume  → ap-south-1a
```

Create the volume.

---

# PART 3: Attach the EBS Volume

Go to:

```text
EBS Volume
    ↓
Actions
    ↓
Attach Volume
    ↓
Select EC2 Server 1
    ↓
Attach
```

Go back to Linux:

```bash
lsblk
```

Example:

```text
nvme0n1      8G
├─nvme0n1p1  /
│
nvme1n1      5G
```

Here:

```text
nvme0n1 → Root Volume
nvme1n1 → New Additional EBS Volume
```

---

# PART 4: Format the New Empty EBS Volume

Because this is a **brand-new empty volume**, create an XFS filesystem:

```bash
sudo mkfs.xfs /dev/nvme1n1
```

⚠️ Run `mkfs.xfs` only for a new empty volume.

---

# PART 5: Create Mount Directory

```bash
sudo mkdir -p /home/ec2-user/logs
```

Mount the volume:

```bash
sudo mount /dev/nvme1n1 /home/ec2-user/logs
```

Check:

```bash
df -h
```

You should see:

```text
/dev/nvme1n1  → /home/ec2-user/logs
```

---

# PART 6: Give Permission and Create Data

The mounted directory may be owned by root.

Check:

```bash
ls -ld /home/ec2-user/logs
```

Give ownership to `ec2-user`:

```bash
sudo chown ec2-user:ec2-user /home/ec2-user/logs
```

Create a test file:

```bash
echo "Hello from Original EBS Volume" > /home/ec2-user/logs/ebslogs.logs
```

Check:

```bash
ls -la /home/ec2-user/logs
```

Read the file:

```bash
cat /home/ec2-user/logs/ebslogs.logs
```

Output:

```text
Hello from Original EBS Volume
```

---

# PART 7: Create Snapshot

Now go to AWS:

```text
AWS Console
    ↓
EC2
    ↓
Volumes
    ↓
Select Original Additional EBS Volume
    ↓
Actions
    ↓
Create Snapshot
```

Wait until the snapshot status becomes:

```text
Completed
```

Your flow is now:

```text
Original EBS Volume
       │
       └── ebslogs.logs
               │
               ▼
            Snapshot
```

---

# PART 8: Create a New Volume from the Snapshot

Go to:

```text
EC2
    ↓
Snapshots
    ↓
Select your Snapshot
    ↓
Actions
    ↓
Create Volume from Snapshot
```

Select the Availability Zone of your **new EC2 Server**.

For example:

```text
New EC2 Server → ap-south-1a
New EBS Volume → ap-south-1a
```

The volume must be in the same AZ as the EC2 instance where you will attach it.

---

# PART 9: Launch EC2 Server 2

Launch a second EC2 instance.

For example:

```text
EC2 Server 1
    ↓
Original EBS Volume

EC2 Server 2
    ↓
New EBS Volume created from Snapshot
```

---

# PART 10: Attach the New Volume to EC2 Server 2

Go to:

```text
New EBS Volume
    ↓
Actions
    ↓
Attach Volume
    ↓
Select EC2 Server 2
```

Login to EC2 Server 2 and check:

```bash
lsblk -f
```

Example:

```text
NAME          FSTYPE MOUNTPOINT
nvme0n1
└─nvme0n1p1   xfs    /

nvme2n1       xfs
```

The device name might be `nvme1n1`, `nvme2n1`, etc. Always verify with:

```bash
lsblk -f
```

---

# PART 11: Create a Mount Directory on EC2 Server 2

```bash
sudo mkdir -p /home/ec2-user/newlogs
```

## Important: Do NOT format this volume

Do **not** run:

```bash
sudo mkfs.xfs /dev/nvme2n1
```

❌ This can destroy the filesystem and data copied from the snapshot.

---

# PART 12: Mount the Volume Created from Snapshot

Because your original volume and cloned volume have the same XFS UUID, use:

```bash
sudo mount -t xfs -o nouuid /dev/nvme2n1 /home/ec2-user/newlogs
```

Then verify:

```bash
df -h
```

Check old data:

```bash
ls -la /home/ec2-user/newlogs
```

Read the file:

```bash
cat /home/ec2-user/newlogs/ebslogs.logs
```

Expected:

```text
Hello from Original EBS Volume
```

---

# Complete Command Workflow

## EC2 Server 1

```bash
# Check disks
lsblk

# Format NEW empty additional volume only
sudo mkfs.xfs /dev/nvme1n1

# Create mount directory
sudo mkdir -p /home/ec2-user/logs

# Mount volume
sudo mount /dev/nvme1n1 /home/ec2-user/logs

# Give permission
sudo chown ec2-user:ec2-user /home/ec2-user/logs

# Create data
echo "Hello from Original EBS Volume" > /home/ec2-user/logs/ebslogs.logs

# Verify
cat /home/ec2-user/logs/ebslogs.logs
ls -la /home/ec2-user/logs
```

### Then in AWS Console

```text
Original EBS Volume
        ↓
Create Snapshot
        ↓
Wait for Completed
        ↓
Create Volume from Snapshot
```

---

## EC2 Server 2

```bash
# Check disks and filesystem
lsblk -f

# Create mount directory
sudo mkdir -p /home/ec2-user/newlogs

# Mount snapshot-created XFS volume
sudo mount -t xfs -o nouuid /dev/nvme2n1 /home/ec2-user/newlogs

# Verify old data
ls -la /home/ec2-user/newlogs

# Read old file
cat /home/ec2-user/newlogs/ebslogs.logs
```

---

# Important Rules

| Scenario                                 | Command                         |
| ---------------------------------------- | ------------------------------- |
| New empty EBS volume                     | `mkfs.xfs` ✅                    |
| Existing volume with data                | `mkfs.xfs` ❌                    |
| Volume created from snapshot             | `mkfs.xfs` ❌                    |
| Original XFS and clone attached together | `mount -o nouuid` may be needed |
| Check disks                              | `lsblk -f`                      |
| Check mount                              | `df -h`                         |
| Check filesystem                         | `blkid`                         |

## Final Architecture

```text
                         AWS REGION
                             │
        ┌────────────────────┴────────────────────┐
        │                                         │
        ▼                                         ▼
   EC2 SERVER 1                              EC2 SERVER 2
        │                                         │
        │                                         │
        ▼                                         ▼
 Original EBS Volume                     Root EBS Volume
        │
        │
        ├── Format XFS (only once)
        │
        ├── Mount /logs
        │
        ├── Create ebslogs.logs
        │
        ▼
     SNAPSHOT
        │
        │ Create Volume
        ▼
 New EBS Volume
        │
        │ Attach to EC2 Server 2
        ▼
 /dev/nvme2n1
        │
        │ mount -o nouuid
        ▼
 /home/ec2-user/newlogs
        │
        ▼
  ebslogs.logs available
```

This is the correct workflow for practicing **EBS → Data → Snapshot → New Volume → New EC2 → Restore Data**.
