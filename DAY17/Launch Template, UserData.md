# Class Brief Explaination
## 1. Launch Template and User Data
Yes 👍 **Launch Template** is the next important AWS topic after AMI.

A **Launch Template** is like a **blueprint for launching EC2 instances**. Instead of selecting AMI, instance type, key pair, security group, storage, etc. every time, you save those settings in one template.

## Simple practice

Let's create a Launch Template using the **AMI you just created**.

### Architecture

```text
AMI
 │
 │
 ▼
Launch Template
 │
 ├── AMI
 ├── Instance Type
 ├── Key Pair
 ├── Security Group
 ├── EBS Volume
 └── User Data
       │
       ▼
   EC2 Instance
```

## Step 1 — Go to Launch Templates

AWS Console:

**EC2 → Launch Templates**

Click:

**Create launch template**

### Step 2 — Give it a name

Use:

```text
Name:
My-WebServer-Launch-Template
```

Description:

```text
Launch template for Apache web server
```

---

## Step 3 — Select your AMI

Under **Application and OS Images (AMI)**:

Select:

**My-WebServer-AMI-Virginia**

This is the AMI you copied from Mumbai to N. Virginia.

So:

```text
AMI:
My-WebServer-AMI-Virginia
```

---

## Step 4 — Select Instance Type

For practice:

```text
t3.micro
```

or another small instance type available to you.

---

## Step 5 — Key Pair

Select your existing key pair.

For example:

```text
MyKeyPair
```

Make sure you actually have the `.pem` private key for this key pair.

---

## Step 6 — Security Group

Select your security group.

For a web-server practice, allow:

```text
SSH
TCP 22
Your IP

HTTP
TCP 80
0.0.0.0/0
```

---

## Step 7 — Storage

You can leave the default storage.

For example:

```text
Root volume
8 GiB
gp3
```

---

# Step 8 — User Data

This is a very useful part of Launch Templates.

You can automatically execute commands when the EC2 starts.

Under **Advanced details → User data**, you can put:

```bash
#!/bin/bash

dnf install httpd -y
systemctl enable httpd
systemctl start httpd

echo "Hello from Launch Template Server" > /var/www/html/index.html
```

However, **because your AMI already contains Apache**, you don't actually need to install Apache again.

For your current AMI practice, you could instead use:

```bash
#!/bin/bash

systemctl enable httpd
systemctl start httpd

echo "Hello from Launch Template Server" > /var/www/html/index.html
```

---

# Step 9 — Create Launch Template

Click:

**Create launch template**

You should see:

```text
My-WebServer-Launch-Template
```

---

# Step 10 — Launch an EC2 from the Launch Template

Go to:

**EC2 → Launch Templates**

Select:

```text
My-WebServer-Launch-Template
```

Click:

**Actions → Launch instance from template**

You don't have to enter all the settings again because they're already stored in the template.

Click:

**Launch instance**

---

# Step 11 — Check your new EC2

After the instance starts, connect:

```bash
ssh -i your-key.pem ec2-user@<PUBLIC-IP>
```

Check:

```bash
sudo systemctl status httpd
```

Then:

```bash
curl localhost
```

You should get:

```text
Hello from Launch Template Server
```

---

# Why Launch Templates are important

Imagine your company needs **10 EC2 servers**.

Without a Launch Template:

```text
Create EC2 #1
Select AMI
Select instance type
Select key
Select SG
Select storage
...
Create EC2 #2
Select AMI
Select instance type
Select key
Select SG
...
```

Very repetitive.

With a Launch Template:

```text
             Launch Template
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼
     EC2-1        EC2-2        EC2-3
       │            │            │
       ▼            ▼            ▼
     EC2-4        EC2-5        EC2-6
```

Every instance can be created with the same configuration.

## 🔥 Very important DevOps connection

This is where AWS concepts start connecting:

```text
AMI
 │
 ▼
Launch Template
 │
 ▼
Auto Scaling Group
 │
 ├── EC2
 ├── EC2
 └── EC2
```

**AMI = server image/template**

**Launch Template = EC2 configuration blueprint**

**Auto Scaling Group = automatically manages multiple EC2 instances**

So after you practice **AMI → Launch Template**, the best next exercise is **Launch Template → Auto Scaling Group**, where AWS automatically launches and replaces EC2 instances.

---
---
# 2. How to Change the default path for httpd
If you mean **change the default Apache HTTPD document root** from:

```text
/var/www/html
```

to another directory, for example:

```text
/home/ec2-user/website
```

here is the step-by-step process for **Amazon Linux**.

### 1. Create the new website directory

```bash
sudo mkdir -p /home/ec2-user/website
or
sudo mkdir -p /home/apache/website
```

Create a test page:

```bash
echo "Hello from New HTTPD Path" | sudo tee /home/ec2-user/website/index.html
```

### 2. Change Apache configuration

Open the Apache configuration:

```bash
sudo vi /etc/httpd/conf/httpd.conf
```

Find:

```apache
DocumentRoot "/var/www/html"
```

Change it to:

```apache
DocumentRoot "/home/ec2-user/website"
or
/home/apache/website
```

Also find:

```apache
<Directory "/var/www/html">
```

Change it to:

```apache
<Directory "/home/ec2-user/website"> or /home/apache/website
or
/home/apache/website
```

Make sure the directory block contains:

```apache
<Directory "/home/ec2-user/website"> or /home/apache/website
    AllowOverride None
    Require all granted
</Directory>
```

Save and exit:

```text
ESC
:wq
ENTER
```

### 3. If using `/home/ec2-user`, fix permissions

Apache needs permission to access the directory:

```bash
sudo chmod 755 /home/ec2-user
sudo chmod 755 /home/ec2-user/website
```

### 4. Test Apache configuration

Before restarting, always check:

```bash
sudo apachectl configtest
```

You want:

```text
Syntax OK
```

### 5. Restart HTTPD

```bash
sudo systemctl restart httpd
```

Check:

```bash
sudo systemctl status httpd
```

### 6. Test

```bash
curl localhost
```

You should get:

```text
Hello from New HTTPD Path
```

Then open:

```text
http://<EC2-PUBLIC-IP>
```

---

### ⭐ Better practice for production

Instead of putting your website inside `/home/ec2-user`, I recommend using something like:

```text
/var/www/mywebsite
```

For example:

```bash
sudo mkdir -p /var/www/mywebsite
echo "My Website" | sudo tee /var/www/mywebsite/index.html
```

Then set:

```apache
DocumentRoot "/var/www/mywebsite"

<Directory "/var/www/mywebsite">
    AllowOverride None
    Require all granted
</Directory>
```

This is cleaner because `/var/www/` is conventionally used for web content.

If you're doing this specifically to practice **EC2 User Data**, I can also give you a User Data script that automatically changes the HTTPD default path when the EC2 launches.

---
---
# 3. How to check the logs directory is 
```
sudo cd /var/log/httpd
sudo cat /var/log/httpd/ access_log
sudo cat /var/log/http/ error_log
```
If you are using **Amazon Linux + Apache (`httpd`)**, there are two main log files.

### 1. Check Apache error logs

```bash
sudo tail -f /var/log/httpd/error_log
```

This continuously shows new errors.

To see the last 50 lines:

```bash
sudo tail -n 50 /var/log/httpd/error_log
```

To see the complete log:

```bash
sudo less /var/log/httpd/error_log
```

---

### 2. Check Apache access logs

This shows requests received by your web server:

```bash
sudo tail -f /var/log/httpd/access_log
```

Last 50 requests:

```bash
sudo tail -n 50 /var/log/httpd/access_log
```

---

### 3. Check whether HTTPD is running

```bash
sudo systemctl status httpd
```

For only recent service logs:

```bash
sudo journalctl -u httpd
```

Live service logs:

```bash
sudo journalctl -u httpd -f
```

---

### 4. Check both logs

```bash
sudo ls -lh /var/log/httpd/
```

Usually you'll see:

```text
access_log
error_log
```

You can also use:

```bash
sudo grep -i error /var/log/httpd/error_log
```

to find errors.

### ⭐ Useful troubleshooting sequence

If your website is not working, run these:

```bash
sudo systemctl status httpd
```

```bash
sudo apachectl configtest
```

```bash
sudo tail -n 50 /var/log/httpd/error_log
```

```bash
sudo tail -n 50 /var/log/httpd/access_log
```

If you want to **create an error intentionally and then see it appear in `error_log`**, I can give you a simple HTTPD practice exercise.
