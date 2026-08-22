# After Rebooting the EC2 Instance it will automatically unmount and again we need to mount the volumes to EC2 Instance but data is safe.

# Class Breif Explaination

1. Create a EC2 Instance and check commands for root volume once lslbk and df -Th and if we want check root info store ls -lrth /dev.
2. Create EBS Volume in GUI with 5GB storage and volume should same with EC2 instance and same Available Zone. If you want create one EBS Volume for Additional volume in different Available Zone, After when attaching to EC2 Instance it will not show to bound the EC2 Instance.
3. Add Tags for Name is Additional Volume, CreatedBy Dhage Anil Kumar and Date, Team and User upto 50 tags we can create.
4. Go to Volume section and select the volume which we created then select Actions later Attach the Volume in same Available Zone, EC2 Instance Name and device name is anything we can go select /dev/sdf.
5. Then go and check in linux lsblk it shows you attached Additional Volume and if we observe here there is no mount any directory.
6. If we go mount file system command df -Th it will not shows 5GB and not mount to any directory.
7. Please check that Additional volume created file system or not by command sudo file -s /dev/nvme1n1(it will different for every ec2 instance).
8. If we got xfs then file system is created and if not then data is not shown.
9. If not showing then create with command sudo mkfs.xfs /dev/nvme1n1 and verify once created or not with command sudo file -s /dev/nvme1n1.
10. Then check once file is mounted or not, it shows no mounted.
11. So that we create mkdir ebsvol and in that create one file called ebs.log some other info and copy the path like /home/ec2-user/ebsvol.
12. Attach one mount point with command sudo mount /dev/nvme1n1 /home/ec2-user/ebsvol and later verify once mounted or not with lsblk and df -Th.
13. Go and reboot once, with command sudo inti 6 and In GUI After Rebooting the EC2 Instance it will automatically unmount and again we need to mount the volumes to EC2 Instance but data is safe.
14. ### Temporary mount

* Stored in `/etc/mtab`
* Lost after reboot
* Again we need to mount for so that we need to run the command is sudo mount /dev/nvme02 /home/ec2-user/ebsvol

15. ### Permanent Mount Configuration

1. Copy the mount entry ---> sudo mount /dev/nvme02 /home/ec2-user/ebsvol.
2. sudo tail -1 /etc/mtab take the info and copy and paste in below command.
3. sudo vi /etc/fstab.
4. once verify the with reboot sudo inti 6 and in GUI reboot, later check with lsblk and df-Th and check with mount -a if not get any issues then fstab is mounted correctly.
5. /etc/mtab  → Shows current mounted filesystems.
   /etc/fstab → Used for permanent mounts after reboot.
16. If we expand the Additional Volume for EBS Volume in GUI with modify 5GB to 9Gb then come and check in Linux lsblk it shows 9GB but in df -Th is shows 5GB only.
17. Then use command sudo xfs_growfs -d /home/ec2-user/ebsvol and later verify with lsblk and df -Th now it shows same 9Gb in both.
18. If we want again expand the volume 9GB to 19Gb it will takes 6hrs to modify.
19. To expand an **AWS EC2 root volume from 8 GB to 9 GB**, there are two steps:

## Step 1: Increase the EBS volume size

First, find the volume ID:

```bash
aws ec2 describe-instances \
  --instance-ids i-xxxxxxxxxxxxxxxxx \
  --query "Reservations[].Instances[].BlockDeviceMappings[].Ebs.VolumeId" \
  --output text
```

Then modify the volume:

```bash
aws ec2 modify-volume \
  --volume-id vol-xxxxxxxxxxxxxxxxx \
  --size 9
```

Check the modification status:

```bash
aws ec2 describe-volumes-modifications \
  --volume-ids vol-xxxxxxxxxxxxxxxxx
```

---

## Step 2: Expand the root partition and filesystem

Check your disk:

```bash
lsblk
```

For example:

```text
NAME         SIZE
nvme0n1        9G
└─nvme0n1p1    8G
```

Install `growpart` if necessary:

```bash
sudo apt update
sudo apt install cloud-guest-utils -y
```

Expand partition 1:

```bash
sudo growpart /dev/nvme0n1 1
```

Then, for an **ext4 filesystem**:

```bash
sudo resize2fs /dev/nvme0n1p1
```

Verify:

```bash
df -h
```

### For older EC2 devices such as `/dev/xvda`

```bash
sudo growpart /dev/xvda 1
sudo resize2fs /dev/xvda1
```

### Simple flow

```text
8 GB EBS Volume
      ↓
AWS CLI modify-volume
      ↓
9 GB EBS Volume
      ↓
growpart
      ↓
Expand root partition
      ↓
resize2fs
      ↓
Root filesystem becomes 9 GB
```

If your filesystem is **XFS** instead of ext4, use `xfs_growfs /` rather than `resize2fs`.

20. In GUI expand Root Volume, goto root vol and select actions, click on modify 8GB to 9GB.
21. Later verify once lsblk and df -Th it shows 8GB only.
22. Run command sudo growpart /dev/nvme0n1 1 and reboot once with sudo inti 6 or do in GUI also and re connect verify once lsblk and df -Th.


# AWS EBS (Elastic Block Store) – Complete In-Depth Guide

---

## 1. What is Amazon EBS?

**Amazon Elastic Block Store (EBS)** is a **block-level storage service** designed to be attached to **EC2 instances**.

Key characteristics:

* Acts like a **virtual hard disk**
* Used for **OS disks and application data**
* Data persists **independently of EC2 lifecycle**
* Can be detached and attached to another instance (within same AZ)

EBS is tightly integrated with EC2 and is one of the **most critical services** in AWS.

---

## 2. How Do We Measure EBS Volume Performance?

EBS performance is evaluated using **four main metrics**.

---

### 2.1 IOPS (Input/Output Operations Per Second)

IOPS represents:

* Number of read/write operations a volume can handle per second

Important:

* Small I/O operations → higher IOPS demand
* Databases are **IOPS-sensitive**

Example:

* 1,000 IOPS = 1,000 read/write operations per second

---

### 2.2 Block Size

Block size is the **amount of data transferred per I/O operation**.

Common block sizes:

* 4 KB
* 8 KB
* 16 KB

Impact:

* Smaller block size → more IOPS
* Larger block size → fewer IOPS, higher throughput

---

### 2.3 Latency

Latency is:

* Time taken to complete **one I/O operation**

Measured in:

* milliseconds (ms)

Lower latency = faster response
Critical for:

* Databases
* Transaction systems
* Real-time applications

---

### 2.4 Throughput

Throughput is:

* Rate at which data is transferred

Measured in:

* MB/s
* GB/s

Important for:

* Large file processing
* Data analytics
* Streaming workloads

---

### 2.5 Relationship Between IOPS, Throughput, and Block Size

A simplified formula:

```
IOPS = (Throughput × 1024) / Block Size (KB)
```

Meaning:

* Increasing block size increases throughput but reduces IOPS
* Workload type decides which metric matters more

---

## 3. EBS Volume Types – High-Level Classification

EBS volumes are broadly classified into:

* **SSD-backed volumes**
* **HDD-backed volumes**
* **Legacy magnetic volumes**

Each type is optimized for **specific workloads**.

---

## 4. Root Volume vs Additional Volumes

### Root Volume

* Contains the **Operating System**
* Mandatory for launching an EC2 instance
* Without root volume → instance cannot boot

Supported root volume types:

* gp2
* gp3
* io1
* io2

---

### Additional (Data) Volumes

* Used for application data, logs, databases
* Can be attached/detached independently

Supported additional volume types:

* gp2, gp3
* io1, io2
* st1, sc1
* standard (magnetic)

---

## 5. SSD-Based EBS Volumes

SSD volumes are optimized for **low latency and random I/O**.

---

### 5.1 General Purpose SSD – gp2 & gp3

These are the **most commonly used EBS volumes**.

#### gp2

* Balanced performance
* Performance scales with size
* Suitable for:

  * Boot volumes
  * Web servers
  * Dev/Test workloads

Size range:

* 1 GB to 16 TB

---

#### gp3 (Recommended)

* Decouples storage size from performance
* You can independently configure:

  * IOPS
  * Throughput

Size range:

* 1 GB to 64 TiB

Why gp3 is better:

* More predictable performance
* Lower cost
* Better control

---

### 5.2 Provisioned IOPS SSD – io1 & io2

Used when **exact IOPS guarantees** are required.

Suitable for:

* Databases
* High-transaction systems
* Mission-critical workloads

---

#### io1

* Provision specific IOPS
* Good for consistent high performance
* Size range: 4 GB to 16 TB

---

#### io2

* Higher durability
* Higher IOPS capability
* Designed for critical production systems

Key characteristics:

* Minimum IOPS: 100
* Maximum IOPS: up to 100,000
* Size range: 4 GB to 64 TiB

---

## 6. HDD-Based EBS Volumes

HDD volumes are optimized for **high throughput**, not random IOPS.

---

### 6.1 Throughput Optimized HDD – st1

Best suited for:

* Big data workloads
* Log processing
* Data warehousing
* Streaming large files

Characteristics:

* High MB/s throughput
* Not suitable for boot volumes
* Lower cost than SSD

Size range:

* 125 GB to 16 TB

---

### 6.2 Cold HDD – sc1

Designed for:

* Infrequently accessed data
* Cold storage
* Cost-sensitive workloads

Characteristics:

* Lowest performance
* Lowest cost
* Sequential access only

Size range:

* 125 GB to 16 TB

---

## 7. Magnetic (Standard) Volumes

Legacy storage option.

Characteristics:

* Lowest cost
* Lowest performance
* Rarely used today

Size range:

* 1 GB to 1 TB

Used only for:

* Legacy systems
* Very low-cost requirements

---

## 8. Availability Zone Limitation (Very Important)

An EBS volume:

* Exists in **one specific Availability Zone**
* Can be attached only to EC2 instances in the **same AZ**

If AZ differs:

* Volume cannot be attached
* Snapshot must be used to move data across AZs

---

## 9. Linux Commands to Work with EBS Volumes

### 9.1 Detect Attached Volumes

```bash
lsblk
```

Shows:

* All block devices detected by OS
* Even unmounted volumes

Example volume names:

* `/dev/nvme0n1` → root volume
* `/dev/nvme1n1` → additional volume

---

### 9.2 View Mounted File Systems

```bash
df -Th
```

Shows:

* Mounted volumes only
* File system type
* Human-readable sizes

---

### 9.3 Check File System Presence

```bash
file -s /dev/nvme1n1
```

Output meaning:

* `data` → no file system
* `XFS` / `ext4` → file system exists

---

## 10. Creating File System and Mounting (Linux)

### Create XFS File System

```bash
mkfs -t xfs /dev/nvme1n1
```

---

### Mount the Volume

```bash
mount /dev/nvme1n1 /home/ec2-user/newvol
```

Temporary mount:

* Stored in `/etc/mtab`
* Lost after reboot

---

### Permanent Mount Configuration

1. Copy mount entry
2. Add it to `/etc/fstab`

Example:

```
mount /dev/nvme1n1 /home/ec2-user/newvol
tail -1 /etc/matb
sudo vi /etc/fstab
```

Activate:

```bash
mount -a
```

---

## 11. Expanding an Existing EBS Volume

### Step 1: Increase Volume Size (AWS Console)

* Modify volume
* Increase size (cannot decrease)

---

### Step 2: Expand File System (Linux – XFS)

```bash
xfs_growfs -d /home/ec2-user/newvol
```

This updates file system to use new space.

---

## 12. Expanding Root Volume (Linux)

### Step 1: Increase root volume size in AWS Console
If increase the volume but not decrease the volume.
one more point to remember is if we increase the volume then again we need to increase the volume then we need to wait for 6hrs.

---

### Step 2: Identify Root Partition

Usually:

* Volume: `/dev/nvme0n1`
* Partition: `1`

---

### Step 3: Grow Partition

```bash
growpart /dev/nvme0n1 1
```

---

### Step 4: Reboot Instance

```bash
reboot
```

File system expands after reboot.

---

## 13. Important Limitation (Must Tell Students)

> **AWS does NOT support reducing EBS volume size.**

If you need smaller size:

* Create new volume
* Copy data
* Replace old volume

---

## 14. Best Practices (Real-World)

* Use **gp3** by default
* Use **io2** for databases
* Never put DB on sc1 or st1
* Keep root volume separate from data
* Always tag volumes
* Take snapshots before resizing
* Monitor IOPS and throughput in CloudWatch

---

## 15. Summary

EBS is not just “disk” — it is:

* Performance-driven
* AZ-scoped
* Workload-specific
* Deeply tied to EC2 behavior

Understanding:

* Volume types
* Performance metrics
* OS-level operations
* Expansion logic

is mandatory for **real AWS engineers**.

