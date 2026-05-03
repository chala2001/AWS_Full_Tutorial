# 🖥️ AWS Full Tutorial — Part 4: EC2 (Elastic Compute Cloud)

> **What is EC2?** EC2 gives you virtual servers (called instances) in the cloud. It's like renting a computer from Amazon that runs 24/7 in their data center.

---

## Table of Contents — Part 4

1. [EC2 Core Concepts](#1-ec2-core-concepts)
2. [Key Pairs (SSH Keys)](#2-key-pairs-ssh-keys)
3. [Launching Your First EC2 Instance](#3-launching-your-first-ec2-instance)
4. [Connecting to EC2 via SSH](#4-connecting-to-ec2-via-ssh)
5. [EC2 Instance Basics (Linux Commands)](#5-ec2-instance-basics-linux-commands)
6. [Installing Software on EC2](#6-installing-software-on-ec2)
7. [Deploying a Web Application on EC2](#7-deploying-a-web-application-on-ec2)
8. [EC2 User Data (Auto-Setup on Launch)](#8-ec2-user-data-auto-setup-on-launch)
9. [EBS Volumes (Storage for EC2)](#9-ebs-volumes-storage-for-ec2)
10. [AMI (Amazon Machine Images)](#10-ami-amazon-machine-images)
11. [EC2 Instance States & Management](#11-ec2-instance-states--management)
12. [Cleanup](#12-cleanup)

---

## 1. EC2 Core Concepts

### 1.1 What You Get with EC2

When you launch an EC2 instance, you get:
- A virtual CPU (vCPU)
- RAM (memory)
- Storage (EBS volume — like a hard drive)
- Network card (with IP address)
- An operating system (Linux or Windows)

### 1.2 Instance Types

| Type | Optimized For | Example | Free Tier |
|------|--------------|---------|-----------|
| **t2.micro** | General purpose (burstable) | Small apps, testing | ✅ YES |
| **t3.micro** | General purpose (burstable, newer) | Small apps, testing | ✅ YES (some regions) |
| **m5.large** | General purpose (steady) | Production apps | ❌ No |
| **c5.large** | Compute optimized | Data processing, ML | ❌ No |
| **r5.large** | Memory optimized | Databases, caching | ❌ No |

> 💡 **We'll use `t2.micro` (1 vCPU, 1 GB RAM) — it's free tier eligible!**

### 1.3 Instance Naming Convention

Example: `t2.micro`
- `t` = Family (T = burstable general purpose)
- `2` = Generation (2nd generation)
- `micro` = Size (smallest)

Sizes from smallest to largest: `nano → micro → small → medium → large → xlarge → 2xlarge → ...`

### 1.4 AMIs (Amazon Machine Images)

An **AMI** is a template that contains the OS and software pre-installed. Think of it as a **template/blueprint** for your server.

| AMI | What It Is |
|-----|-----------|
| **Amazon Linux 2023** | AWS's own Linux (recommended, free) |
| **Ubuntu Server 22.04 LTS** | Popular Linux distribution (free) |
| **Windows Server 2022** | Microsoft Windows (free tier eligible) |
| **Red Hat Enterprise Linux** | Enterprise Linux (may cost extra) |

> 💡 **We'll use Amazon Linux 2023 — it's optimized for AWS and free.**

### 1.5 Free Tier Limits for EC2

| What | Limit |
|------|-------|
| Instance type | `t2.micro` only (or `t3.micro` in some regions) |
| Hours per month | 750 hours (enough for 1 instance running 24/7) |
| Storage | 30 GB of EBS (gp2 or gp3) |
| Duration | First 12 months after account creation |

---

## 2. Key Pairs (SSH Keys)

### 2.1 What is a Key Pair?

A key pair is used to **securely connect to your EC2 instance**. It consists of:
- **Public key** — stored on the EC2 instance (like a lock)
- **Private key** — stored on YOUR computer (like a key)

Only someone with the private key can unlock (SSH into) the instance.

### 2.2 Create a Key Pair

**Console:**

1. Go to **EC2** dashboard (search "EC2" in search bar)
2. Left sidebar → **"Key Pairs"** (under Network & Security)
3. Click **"Create key pair"**
4. **Name:** `my-aws-keypair`
5. **Key pair type:** RSA
6. **Private key file format:** `.pem` (for SSH from PowerShell/Git Bash)
7. Click **"Create key pair"**
8. The `.pem` file will **automatically download**
9. **SAVE IT SECURELY!** Move it to: `C:\Users\ASUS\.ssh\my-aws-keypair.pem`

**Create the .ssh folder if it doesn't exist:**
```powershell
mkdir C:\Users\ASUS\.ssh -ErrorAction SilentlyContinue
Move-Item C:\Users\ASUS\Downloads\my-aws-keypair.pem C:\Users\ASUS\.ssh\my-aws-keypair.pem
```

> ⚠️ **You can NEVER download this file again!** If you lose it, you'll need to create a new key pair.

**CLI:**
```powershell
aws ec2 create-key-pair --key-name my-aws-keypair --query "KeyMaterial" --output text > C:\Users\ASUS\.ssh\my-aws-keypair.pem
```

---

## 3. Launching Your First EC2 Instance

### 3.1 Using the Default VPC (Quick Start)

For your first time, let's use the default VPC. Later we'll use our custom VPC.

**Console Steps:**

1. Go to **EC2** → Click **"Launch instances"** (orange button)

2. **Name and tags:**
   - **Name:** `my-first-server`

3. **Application and OS Images (AMI):**
   - Select **"Amazon Linux 2023 AMI"** (should be the first option)
   - Architecture: **64-bit (x86)** ← Keep default
   - It should say **"Free tier eligible"** ✅

4. **Instance type:**
   - Select **`t2.micro`** (Free tier eligible)
   - 1 vCPU, 1 GiB Memory

5. **Key pair:**
   - Select **`my-aws-keypair`** (the one you created)

6. **Network settings:** Click **"Edit"**
   - **VPC:** Default VPC (or select `my-custom-vpc` if you want)
   - **Subnet:** If using default VPC, select any subnet
   - **Auto-assign public IP:** Enable
   - **Security group:** Select **"Create security group"**
     - **Security group name:** `my-first-server-sg`
     - **Description:** `Allow SSH and HTTP`
     - **Rule 1:** SSH, Port 22, Source: **My IP**
     - Click **"Add security group rule"**
     - **Rule 2:** HTTP, Port 80, Source: **Anywhere (0.0.0.0/0)**
     - Click **"Add security group rule"**
     - **Rule 3:** Custom TCP, Port 8080, Source: **Anywhere (0.0.0.0/0)**

7. **Configure storage:**
   - **Size:** `8` GiB (default, free tier allows up to 30 GiB)
   - **Volume type:** gp3 (General Purpose SSD)
   - Keep **"Delete on termination"** checked ✅

8. **Advanced details:**
   - Leave everything as default for now (we'll use User Data later)

9. **Summary (right side):**
   - Number of instances: `1`
   - Review everything looks correct

10. Click **"Launch instance"** 🚀

11. You'll see "Successfully initiated launch of instance"
12. Click on the instance ID link (e.g., `i-0abc123def456`)

### 3.2 Wait for Instance to Start

1. You're now on the **Instances** page
2. Wait for:
   - **Instance state:** `Running` ✅ (green)
   - **Status checks:** `2/2 checks passed` ✅
3. This takes about 1-2 minutes

### 3.3 Find Your Instance's Public IP

1. Click on your instance
2. In the **Details** tab, find:
   - **Public IPv4 address:** e.g., `13.232.45.67`
   - **Public IPv4 DNS:** e.g., `ec2-13-232-45-67.ap-south-1.compute.amazonaws.com`

> **Save this IP!** You'll need it to SSH.

---

## 4. Connecting to EC2 via SSH

### 4.1 Method 1: Using PowerShell (Windows 10/11)

```powershell
ssh -i C:\Users\ASUS\.ssh\my-aws-keypair.pem ec2-user@YOUR_PUBLIC_IP
```

Replace `YOUR_PUBLIC_IP` with your instance's public IP. Example:
```powershell
ssh -i C:\Users\ASUS\.ssh\my-aws-keypair.pem ec2-user@13.232.45.67
```

**First time connecting?** You'll see:
```
The authenticity of host '13.232.45.67' can't be established.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```
Type `yes` and press Enter.

**If you get a "permissions are too open" error:**
```powershell
# Fix file permissions on Windows
icacls C:\Users\ASUS\.ssh\my-aws-keypair.pem /inheritance:r
icacls C:\Users\ASUS\.ssh\my-aws-keypair.pem /grant:r "%USERNAME%:R"
```

Then try SSH again.

### 4.2 Method 2: EC2 Instance Connect (Browser-based)

1. Go to EC2 → Instances → Select your instance
2. Click **"Connect"** (top button)
3. Select **"EC2 Instance Connect"** tab
4. Click **"Connect"**
5. A terminal opens right in your browser! 🎉

### 4.3 Method 3: Using Git Bash

If you have Git installed:
1. Open **Git Bash**
2. Run:
```bash
ssh -i /c/Users/ASUS/.ssh/my-aws-keypair.pem ec2-user@YOUR_PUBLIC_IP
```

### 4.4 You're In! 🎉

You should see something like:
```
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\       AL2023
  ~~     \###|
  ~~       \#/ ___
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'
[ec2-user@ip-172-31-10-5 ~]$
```

**You are now inside your EC2 instance!** This is a Linux server running in AWS.

---

## 5. EC2 Instance Basics (Linux Commands)

Now that you're SSH'd in, let's learn some essential Linux commands:

### 5.1 Navigation

```bash
# Where am I?
pwd
# Output: /home/ec2-user

# List files
ls -la

# Go to root directory
cd /

# List everything in root
ls

# Go back to home
cd ~
```

### 5.2 System Info

```bash
# What OS am I running?
cat /etc/os-release

# How much RAM do I have?
free -h

# How much disk space?
df -h

# How many CPUs?
nproc

# Who am I logged in as?
whoami
# Output: ec2-user
```

### 5.3 File Operations

```bash
# Create a file
echo "Hello from EC2!" > hello.txt

# View the file
cat hello.txt

# Create a directory
mkdir my-project

# Move into it
cd my-project

# Create another file
touch index.html
echo "<h1>Hello from EC2</h1>" > index.html
```

### 5.4 Super User (sudo)

`ec2-user` is not root, but can run commands AS root using `sudo`:

```bash
# Update all packages (like Windows Update)
sudo yum update -y

# Install something
sudo yum install -y tree
```

---

## 6. Installing Software on EC2

### 6.1 Update the System

Always update first:
```bash
sudo yum update -y
```

### 6.2 Install Git

```bash
sudo yum install -y git
git --version
```

### 6.3 Install Node.js (for web apps)

```bash
# Install Node.js via nvm (Node Version Manager)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# Reload shell
source ~/.bashrc

# Install latest LTS version
nvm install --lts

# Verify
node --version
npm --version
```

### 6.4 Install Java

```bash
# Install Java 17
sudo yum install -y java-17-amazon-corretto

# Verify
java -version
```

### 6.5 Install Python 3

```bash
# Python 3 is pre-installed on Amazon Linux 2023
python3 --version

# Install pip
sudo yum install -y python3-pip
pip3 --version
```

### 6.6 Install Docker

```bash
# Install Docker
sudo yum install -y docker

# Start Docker service
sudo systemctl start docker

# Enable Docker to start on boot
sudo systemctl enable docker

# Add ec2-user to docker group (so you don't need sudo)
sudo usermod -aG docker ec2-user

# Log out and back in for group change to take effect
exit
```

Then SSH back in:
```powershell
ssh -i C:\Users\ASUS\.ssh\my-aws-keypair.pem ec2-user@YOUR_PUBLIC_IP
```

```bash
# Verify Docker
docker --version
docker ps
```

---

## 7. Deploying a Web Application on EC2

### 7.1 Deploy a Simple Node.js App

**On your EC2 instance:**

```bash
# Create project directory
mkdir ~/my-web-app && cd ~/my-web-app

# Initialize Node.js project
npm init -y

# Install Express.js
npm install express

# Create the app
cat > app.js << 'EOF'
const express = require('express');
const app = express();
const PORT = 8080;

app.get('/', (req, res) => {
    res.send(`
        <html>
        <head><title>My EC2 App</title></head>
        <body style="font-family: Arial; background: #1a1a2e; color: white;
                     display: flex; justify-content: center; align-items: center;
                     height: 100vh; margin: 0;">
            <div style="text-align: center; padding: 40px;
                        background: rgba(255,255,255,0.1); border-radius: 20px;">
                <h1 style="color: #e94560;">🚀 Hello from AWS EC2!</h1>
                <p>This app is running on an EC2 instance.</p>
                <p>Instance Region: ap-south-1</p>
                <p>Server Time: ${new Date().toISOString()}</p>
            </div>
        </body>
        </html>
    `);
});

app.get('/health', (req, res) => {
    res.json({ status: 'healthy', uptime: process.uptime() });
});

app.listen(PORT, '0.0.0.0', () => {
    console.log(`Server running on port ${PORT}`);
});
EOF

# Run the app
node app.js
```

**Now open your browser and go to:**
```
http://YOUR_PUBLIC_IP:8080
```

🎉 **Your web app is live on the internet!**

Press `Ctrl+C` to stop the app.

### 7.2 Run App in Background (Using PM2)

```bash
# Install PM2 (process manager)
npm install -g pm2

# Start app with PM2
pm2 start app.js --name my-web-app

# Check status
pm2 status

# View logs
pm2 logs

# Auto-start PM2 on reboot
pm2 startup
# Run the command it outputs (starts with sudo)
pm2 save
```

Now the app keeps running even after you close your SSH session!

**PM2 Commands:**
```bash
pm2 stop my-web-app     # Stop the app
pm2 restart my-web-app   # Restart the app
pm2 delete my-web-app    # Remove from PM2
pm2 list                 # List all apps
pm2 monit                # Real-time monitoring
```

---

## 8. EC2 User Data (Auto-Setup on Launch)

### 8.1 What is User Data?

**User Data** is a script that runs **automatically when an EC2 instance first launches**. It's like telling the instance: "When you start, do these things."

### 8.2 Example User Data Script

This script automatically installs Node.js, creates an app, and starts it:

```bash
#!/bin/bash
yum update -y
yum install -y git

# Install Node.js
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
nvm install --lts

# Create app
mkdir -p /home/ec2-user/app
cd /home/ec2-user/app
cat > app.js << 'APPEOF'
const http = require('http');
const server = http.createServer((req, res) => {
    res.writeHead(200, {'Content-Type': 'text/html'});
    res.end('<h1>Auto-deployed with User Data!</h1>');
});
server.listen(8080, () => console.log('Running on 8080'));
APPEOF

chown -R ec2-user:ec2-user /home/ec2-user/app
su - ec2-user -c "cd /home/ec2-user/app && node app.js &"
```

### 8.3 How to Add User Data

When launching an instance (Step 8 of the launch wizard):

1. Expand **"Advanced details"**
2. Scroll down to **"User data"**
3. Paste your script in the text box
4. Launch the instance

The script runs as `root` automatically on first boot.

---

## 9. EBS Volumes (Storage for EC2)

### 9.1 What is EBS?

**EBS (Elastic Block Store)** is the hard drive/SSD attached to your EC2 instance. Every EC2 instance needs at least one EBS volume (the root volume with the OS).

### 9.2 EBS Volume Types

| Type | Name | Use Case | Free Tier |
|------|------|----------|-----------|
| `gp3` | General Purpose SSD | Most workloads | ✅ 30 GB free |
| `gp2` | General Purpose SSD (older) | Most workloads | ✅ 30 GB free |
| `io1/io2` | Provisioned IOPS SSD | High-performance databases | ❌ |
| `st1` | Throughput Optimized HDD | Big data, data warehouses | ❌ |
| `sc1` | Cold HDD | Infrequent access | ❌ |

### 9.3 Create and Attach an Additional EBS Volume

**Console:**

1. Go to **EC2** → Left sidebar → **"Volumes"** (under Elastic Block Store)
2. Click **"Create volume"**
3. **Volume type:** gp3
4. **Size:** 5 GiB
5. **Availability Zone:** Same as your EC2 instance! (e.g., `ap-south-1a`)
   - ⚠️ Must be same AZ as your instance!
6. Click **"Create volume"**

**Attach to Instance:**

7. Select the new volume → **Actions** → **"Attach volume"**
8. **Instance:** Select your running instance
9. **Device name:** `/dev/xvdf` (default)
10. Click **"Attach volume"**

**SSH into your instance and mount the volume:**

```bash
# List block devices
lsblk

# Create a filesystem on the new volume
sudo mkfs -t ext4 /dev/xvdf

# Create mount point
sudo mkdir /data

# Mount the volume
sudo mount /dev/xvdf /data

# Verify
df -h

# Make it persist across reboots
echo '/dev/xvdf /data ext4 defaults,nofail 0 2' | sudo tee -a /etc/fstab
```

### 9.4 EBS Snapshots (Backups)

A snapshot is a backup of your EBS volume at a point in time.

```bash
# Create a snapshot (CLI, run from Windows PowerShell)
aws ec2 create-snapshot --volume-id vol-0abc123 --description "My first snapshot"
```

**Console:**
1. EC2 → Volumes → Select volume → Actions → Create snapshot
2. Add description → Create snapshot

> 💡 30 GB of snapshot storage is free. Delete snapshots you don't need!

---

## 10. AMI (Amazon Machine Images)

### 10.1 What is an AMI?

An AMI is a **snapshot of your entire EC2 instance** — OS, installed software, configurations, everything. You can use it to launch new identical instances.

### 10.2 Create Your Own AMI

**Why?** You installed Node.js, Docker, your app. Save all that as an AMI so you can launch a pre-configured server anytime.

**Console:**

1. EC2 → Instances → Select your instance
2. **Actions** → **Image and templates** → **Create image**
3. **Image name:** `my-webserver-ami-2026`
4. **Image description:** `Amazon Linux 2023 with Node.js, Docker, and my web app`
5. Keep default settings
6. Click **"Create image"**

Wait a few minutes for the AMI to be created.

**To use it later:**
1. EC2 → Launch instances
2. In AMI selection → **"My AMIs"** tab
3. Select `my-webserver-ami-2026`
4. Launch as normal!

---

## 11. EC2 Instance States & Management

### 11.1 Instance States

| State | Meaning | Billing |
|-------|---------|---------|
| **Running** | Instance is on and working | ✅ You're being charged hours |
| **Stopped** | Instance is off (like shutting down a PC) | ❌ No compute charges (but EBS storage still charged) |
| **Terminated** | Instance is deleted permanently | ❌ No charges |
| **Pending** | Instance is starting up | Transitional |
| **Stopping** | Instance is shutting down | Transitional |

### 11.2 Managing Instances

**Console:**
- Select instance → **Instance state** dropdown:
  - **Stop instance** — Shut it down (can restart later)
  - **Start instance** — Turn it back on
  - **Reboot instance** — Restart it
  - **Terminate instance** — Delete it permanently

**CLI:**
```powershell
# Stop
aws ec2 stop-instances --instance-ids i-0abc123def456

# Start
aws ec2 start-instances --instance-ids i-0abc123def456

# Terminate (delete)
aws ec2 terminate-instances --instance-ids i-0abc123def456
```

### 11.3 ⚠️ Important Notes

- **Stop vs Terminate:**
  - **Stop** = You can start it again later. Like hibernating your laptop.
  - **Terminate** = GONE FOREVER. Like throwing your laptop in a volcano.
- When you **stop** and **start**, the public IP changes! (Unless you use an Elastic IP)
- **EBS charges continue** even when instance is stopped (unless you delete the volume)

---

## 12. Cleanup

### ⚠️ Do this when you're done practicing!

```powershell
# Terminate the instance
aws ec2 terminate-instances --instance-ids i-YOUR_INSTANCE_ID

# Delete key pair (optional, only if you want to)
aws ec2 delete-key-pair --key-name my-aws-keypair

# Check for any running instances
aws ec2 describe-instances --query "Reservations[].Instances[?State.Name=='running'].InstanceId" --output text
```

**Console:**
1. EC2 → Instances → Select all instances → Instance state → **Terminate instance**
2. EC2 → Volumes → Delete any additional volumes
3. EC2 → Snapshots → Delete any snapshots
4. EC2 → AMIs → Deregister any custom AMIs
5. EC2 → Security Groups → Delete non-default security groups
6. EC2 → Elastic IPs → Release any Elastic IPs

---

## ✅ Part 4 Checklist

- [ ] Understand EC2 instance types (t2.micro = free)
- [ ] Created a key pair
- [ ] Launched an EC2 instance
- [ ] Connected via SSH from Windows
- [ ] Learned basic Linux commands
- [ ] Installed Node.js, Git, Docker on EC2
- [ ] Deployed a web application on EC2
- [ ] Used PM2 for process management
- [ ] Understand User Data for auto-setup
- [ ] Understand EBS volumes and snapshots
- [ ] Created a custom AMI
- [ ] Know the difference between Stop vs Terminate
- [ ] Cleaned up all resources

---

> 📖 **Next: Part 5 — Docker on EC2 & ECR** — Containerize your app with Docker and push images to AWS's container registry!
