# 🌐 AWS Full Tutorial — Part 3: VPC, Subnets & Networking

> **Why Networking First?** Before launching any EC2 server, you MUST understand networking. Every EC2 instance lives inside a VPC and subnet. Security groups control what traffic can reach it.

---

## Table of Contents — Part 3

1. [Networking Concepts (Theory)](#1-networking-concepts-theory)
2. [VPC — Virtual Private Cloud](#2-vpc--virtual-private-cloud)
3. [Subnets — Public vs Private](#3-subnets--public-vs-private)
4. [Internet Gateway (IGW)](#4-internet-gateway-igw)
5. [Route Tables](#5-route-tables)
6. [Security Groups (Firewall)](#6-security-groups-firewall)
7. [Network ACLs (NACLs)](#7-network-acls-nacls)
8. [Elastic IPs](#8-elastic-ips)
9. [Hands-On: Build a Custom VPC from Scratch](#9-hands-on-build-a-custom-vpc-from-scratch)
10. [Cleanup](#10-cleanup)

---

## 1. Networking Concepts (Theory)

### 1.1 IP Addresses

Every device on a network has an IP address (like a home address for your computer).

| Type | Example | Where |
|------|---------|-------|
| **Public IP** | `54.123.45.67` | Accessible from the internet |
| **Private IP** | `10.0.1.15` | Only accessible within your network |

**Private IP Ranges (RFC 1918):**
| Range | Usage |
|-------|-------|
| `10.0.0.0 — 10.255.255.255` | Large networks |
| `172.16.0.0 — 172.31.255.255` | Medium networks |
| `192.168.0.0 — 192.168.255.255` | Home/small networks |

### 1.2 CIDR Notation

CIDR (Classless Inter-Domain Routing) is how we define network ranges.

| CIDR | Meaning | Number of IPs |
|------|---------|--------------|
| `10.0.0.0/32` | Just one IP: 10.0.0.0 | 1 |
| `10.0.0.0/24` | 10.0.0.0 to 10.0.0.255 | 256 |
| `10.0.0.0/16` | 10.0.0.0 to 10.0.255.255 | 65,536 |
| `10.0.0.0/8` | 10.0.0.0 to 10.255.255.255 | 16,777,216 |
| `0.0.0.0/0` | ALL IP addresses (the entire internet) | All |

**How to read it:** The number after `/` is how many bits are fixed.
- `/24` = first 24 bits fixed = first 3 octets fixed = last octet varies (256 IPs)
- `/16` = first 16 bits fixed = first 2 octets fixed = last 2 vary (65,536 IPs)

### 1.3 Ports

Ports are like doors on a server. Each service uses a specific port.

| Port | Service |
|------|---------|
| 22 | SSH (secure shell — remote login to Linux) |
| 80 | HTTP (web traffic, unencrypted) |
| 443 | HTTPS (web traffic, encrypted) |
| 3389 | RDP (Remote Desktop — Windows) |
| 3000 | Common for Node.js apps |
| 8080 | Common alternative for web apps |
| 5432 | PostgreSQL database |
| 3306 | MySQL database |
| 27017 | MongoDB |

---

## 2. VPC — Virtual Private Cloud

### 2.1 What is a VPC?

A **VPC** is your **own private network** within AWS. It's like having your own isolated section of the AWS cloud.

Think of it as your **own building** in a massive apartment complex (AWS). Only you have the keys. You decide who can enter and what rooms (subnets) exist.

### 2.2 Key VPC Facts

- Every AWS account comes with a **default VPC** in each region
- You can create **custom VPCs** (which we'll do)
- A VPC spans **one Region** but can span **multiple Availability Zones**
- You define the IP range using CIDR notation (e.g., `10.0.0.0/16`)
- VPCs are **FREE** — no charge for creating one!

### 2.3 Default VPC vs Custom VPC

| Feature | Default VPC | Custom VPC |
|---------|-------------|------------|
| Created automatically | Yes (one per region) | No (you create it) |
| CIDR block | `172.31.0.0/16` | You choose |
| Internet access | Yes (public subnets) | You configure |
| Use case | Quick testing | Production workloads |

---

## 3. Subnets — Public vs Private

### 3.1 What is a Subnet?

A **subnet** is a **sub-section** of your VPC. It's like dividing your building into rooms.

Each subnet lives in **one Availability Zone** and has its own IP range (a subset of the VPC's range).

### 3.2 Public vs Private Subnets

| Feature | Public Subnet | Private Subnet |
|---------|--------------|----------------|
| Internet access | YES (via Internet Gateway) | NO direct internet access |
| Use case | Web servers, load balancers | Databases, app servers |
| Route table | Has route to Internet Gateway | No route to Internet Gateway |

### 3.3 Example Architecture

```
VPC: 10.0.0.0/16 (65,536 IPs)
│
├── Public Subnet A (AZ-a): 10.0.1.0/24 (256 IPs)
│   └── Web Server (EC2) — accessible from internet
│
├── Public Subnet B (AZ-b): 10.0.2.0/24 (256 IPs)
│   └── Web Server (EC2) — accessible from internet
│
├── Private Subnet A (AZ-a): 10.0.3.0/24 (256 IPs)
│   └── Database (RDS) — NOT accessible from internet
│
└── Private Subnet B (AZ-b): 10.0.4.0/24 (256 IPs)
    └── Database (RDS) — NOT accessible from internet
```

---

## 4. Internet Gateway (IGW)

### 4.1 What is an Internet Gateway?

An **Internet Gateway** is the door between your VPC and the internet.

Without an IGW, nothing in your VPC can access the internet, and no one from the internet can access your VPC.

### 4.2 Key Facts

- One IGW per VPC
- FREE (no charges)
- Must be attached to a VPC
- Must be referenced in a route table for subnets that need internet access

---

## 5. Route Tables

### 5.1 What is a Route Table?

A **route table** contains rules (called routes) that determine where network traffic goes.

Think of it as a **GPS** for your VPC — it tells traffic which direction to go.

### 5.2 Example Route Table for Public Subnet

| Destination | Target | Meaning |
|-------------|--------|---------|
| `10.0.0.0/16` | local | Traffic within the VPC stays local |
| `0.0.0.0/0` | igw-xxxxx | All other traffic goes to the Internet Gateway |

### 5.3 Example Route Table for Private Subnet

| Destination | Target | Meaning |
|-------------|--------|---------|
| `10.0.0.0/16` | local | Traffic within the VPC stays local |

> Notice: No route to `0.0.0.0/0` — so this subnet has NO internet access.

---

## 6. Security Groups (Firewall)

### 6.1 What is a Security Group?

A **Security Group** is a **virtual firewall** that controls inbound and outbound traffic for your EC2 instances.

Think of it as a **bouncer at a club** — decides who gets in and who gets out.

### 6.2 Key Facts

| Feature | Detail |
|---------|--------|
| **Level** | Instance level (attached to individual EC2 instances) |
| **Default behavior** | Blocks ALL inbound, allows ALL outbound |
| **Rules** | Only ALLOW rules (no deny rules) |
| **Stateful** | If inbound traffic is allowed, the response is automatically allowed |
| **Can reference** | Other security groups (not just IPs) |

### 6.3 Common Security Group Rules

**For a Web Server (EC2):**
| Type | Protocol | Port | Source | Why |
|------|----------|------|--------|-----|
| SSH | TCP | 22 | Your IP (e.g., `103.x.x.x/32`) | So you can SSH into the server |
| HTTP | TCP | 80 | `0.0.0.0/0` | So anyone can visit your website |
| HTTPS | TCP | 443 | `0.0.0.0/0` | Secure web traffic |
| Custom TCP | TCP | 8080 | `0.0.0.0/0` | For apps running on port 8080 |

**For a Database (RDS):**
| Type | Protocol | Port | Source | Why |
|------|----------|------|--------|-----|
| MySQL/Aurora | TCP | 3306 | `sg-webserver` | Only web server's security group can access |

> 💡 **Best Practice:** Never open SSH (port 22) to `0.0.0.0/0` (entire internet). Always restrict to YOUR IP.

### 6.4 How to Find Your IP

Open your browser and go to: [https://checkip.amazonaws.com](https://checkip.amazonaws.com)

It will show something like: `103.50.123.45`

Your CIDR: `103.50.123.45/32` (the `/32` means just this one IP)

---

## 7. Network ACLs (NACLs)

### 7.1 What is a NACL?

A **Network ACL** is a firewall at the **subnet level** (not instance level like security groups).

### 7.2 Security Groups vs NACLs

| Feature | Security Group | NACL |
|---------|---------------|------|
| **Level** | Instance | Subnet |
| **Rules** | Allow only | Allow AND Deny |
| **Stateful?** | Yes (return traffic auto-allowed) | No (must explicitly allow return traffic) |
| **Rule evaluation** | All rules evaluated | Rules evaluated in order (by number) |
| **Default** | Deny all inbound, allow all outbound | Allow all inbound and outbound |

> 💡 For most use cases, **Security Groups are sufficient**. NACLs add an extra layer.

---

## 8. Elastic IPs

### 8.1 What is an Elastic IP?

An **Elastic IP** is a **static, public IPv4 address** that you can allocate and attach to any EC2 instance.

**Why do you need one?**
- When you stop and start an EC2 instance, its public IP changes
- An Elastic IP stays the same even after stopping/starting

### 8.2 Important Cost Warning!

| Situation | Cost |
|-----------|------|
| Elastic IP attached to a **running** EC2 instance | **FREE** |
| Elastic IP **NOT attached** to any instance | **$0.005/hour (~$3.60/month)** |
| Elastic IP attached to a **stopped** instance | **$0.005/hour** |

> ⚠️ **Always release Elastic IPs you're not using!**

---

## 9. Hands-On: Build a Custom VPC from Scratch

### 9.1 What We'll Build

```
Custom VPC (10.0.0.0/16)
├── Internet Gateway
├── Public Subnet (10.0.1.0/24) in AZ-a
│   ├── Route Table → Internet Gateway
│   └── (We'll put our EC2 here in Part 4)
├── Private Subnet (10.0.2.0/24) in AZ-b
│   └── Route Table → Local only
└── Security Group for Web Server
```

### 9.2 Step 1: Create the VPC

**Console Method:**

1. Search for **"VPC"** → Click **VPC**
2. Click **"Create VPC"**
3. Select **"VPC only"** (not "VPC and more" — we'll create subnets manually to learn)
4. **Name tag:** `my-custom-vpc`
5. **IPv4 CIDR block:** `10.0.0.0/16`
6. **IPv6 CIDR block:** No IPv6 CIDR block
7. **Tenancy:** Default
8. Click **"Create VPC"**

**CLI Method:**
```powershell
aws ec2 create-vpc --cidr-block 10.0.0.0/16 --tag-specifications "ResourceType=vpc,Tags=[{Key=Name,Value=my-custom-vpc}]"
```

> Save the **VPC ID** (e.g., `vpc-0abc123def456`). You'll need it for the next steps.

### 9.3 Step 2: Enable DNS Hostnames

By default, custom VPCs don't assign DNS hostnames to instances.

**Console:**
1. Select your VPC → **Actions** → **Edit VPC settings**
2. Check ✅ **"Enable DNS hostnames"**
3. Click **"Save"**

**CLI:**
```powershell
aws ec2 modify-vpc-attribute --vpc-id vpc-0abc123def456 --enable-dns-hostnames "{\"Value\":true}"
```

### 9.4 Step 3: Create an Internet Gateway

**Console:**
1. In VPC dashboard, left sidebar → **"Internet gateways"**
2. Click **"Create internet gateway"**
3. **Name tag:** `my-igw`
4. Click **"Create internet gateway"**
5. Click **"Actions"** → **"Attach to VPC"**
6. Select `my-custom-vpc` → Click **"Attach internet gateway"**

**CLI:**
```powershell
# Create IGW
aws ec2 create-internet-gateway --tag-specifications "ResourceType=internet-gateway,Tags=[{Key=Name,Value=my-igw}]"

# Attach to VPC (use your actual IDs)
aws ec2 attach-internet-gateway --internet-gateway-id igw-0abc123 --vpc-id vpc-0abc123def456
```

### 9.5 Step 4: Create Subnets

**Public Subnet:**

1. Left sidebar → **"Subnets"** → **"Create subnet"**
2. **VPC ID:** Select `my-custom-vpc`
3. **Subnet name:** `my-public-subnet`
4. **Availability Zone:** Select the first AZ (e.g., `ap-south-1a`)
5. **IPv4 CIDR block:** `10.0.1.0/24`
6. Click **"Create subnet"**

**Private Subnet:**

7. Click **"Create subnet"** again
8. **VPC ID:** Select `my-custom-vpc`
9. **Subnet name:** `my-private-subnet`
10. **Availability Zone:** Select a different AZ (e.g., `ap-south-1b`)
11. **IPv4 CIDR block:** `10.0.2.0/24`
12. Click **"Create subnet"**

**Enable Auto-assign Public IP for Public Subnet:**

13. Select `my-public-subnet` → **Actions** → **"Edit subnet settings"**
14. Check ✅ **"Enable auto-assign public IPv4 address"**
15. Click **"Save"**

**CLI:**
```powershell
# Public subnet
aws ec2 create-subnet --vpc-id vpc-0abc123def456 --cidr-block 10.0.1.0/24 --availability-zone ap-south-1a --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=my-public-subnet}]"

# Private subnet
aws ec2 create-subnet --vpc-id vpc-0abc123def456 --cidr-block 10.0.2.0/24 --availability-zone ap-south-1b --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=my-private-subnet}]"

# Enable auto-assign public IP
aws ec2 modify-subnet-attribute --subnet-id subnet-0abc123 --map-public-ip-on-launch
```

### 9.6 Step 5: Create Route Tables

**Public Route Table:**

1. Left sidebar → **"Route tables"**
2. Click **"Create route table"**
3. **Name:** `my-public-rt`
4. **VPC:** `my-custom-vpc`
5. Click **"Create route table"**

**Add Internet Route:**

6. Select `my-public-rt` → **"Routes"** tab → **"Edit routes"**
7. Click **"Add route"**
   - **Destination:** `0.0.0.0/0`
   - **Target:** Select **"Internet Gateway"** → Select `my-igw`
8. Click **"Save changes"**

**Associate with Public Subnet:**

9. Go to **"Subnet associations"** tab → **"Edit subnet associations"**
10. Check ✅ `my-public-subnet`
11. Click **"Save associations"**

**CLI:**
```powershell
# Create route table
aws ec2 create-route-table --vpc-id vpc-0abc123def456 --tag-specifications "ResourceType=route-table,Tags=[{Key=Name,Value=my-public-rt}]"

# Add internet route
aws ec2 create-route --route-table-id rtb-0abc123 --destination-cidr-block 0.0.0.0/0 --gateway-id igw-0abc123

# Associate with subnet
aws ec2 associate-route-table --route-table-id rtb-0abc123 --subnet-id subnet-0abc123
```

> The private subnet will use the **main route table** (which only has the local route). No changes needed.

### 9.7 Step 6: Create a Security Group

1. Left sidebar → **"Security groups"** → **"Create security group"**
2. **Security group name:** `web-server-sg`
3. **Description:** `Allow SSH and HTTP access`
4. **VPC:** Select `my-custom-vpc`

**Inbound Rules (Click "Add rule" for each):**

| Type | Port Range | Source | Description |
|------|-----------|--------|-------------|
| SSH | 22 | My IP (dropdown selects your current IP automatically) | SSH access from my laptop |
| HTTP | 80 | Anywhere-IPv4 (`0.0.0.0/0`) | Web traffic from anywhere |
| HTTPS | 443 | Anywhere-IPv4 (`0.0.0.0/0`) | Secure web traffic |
| Custom TCP | 8080 | Anywhere-IPv4 (`0.0.0.0/0`) | App server port |

**Outbound Rules:** Leave as default (all traffic allowed outbound)

5. Click **"Create security group"**

**CLI:**
```powershell
# Create security group
aws ec2 create-security-group --group-name web-server-sg --description "Allow SSH and HTTP" --vpc-id vpc-0abc123def456

# Add SSH rule (replace YOUR_IP)
aws ec2 authorize-security-group-ingress --group-id sg-0abc123 --protocol tcp --port 22 --cidr YOUR_IP/32

# Add HTTP rule
aws ec2 authorize-security-group-ingress --group-id sg-0abc123 --protocol tcp --port 80 --cidr 0.0.0.0/0

# Add HTTPS rule
aws ec2 authorize-security-group-ingress --group-id sg-0abc123 --protocol tcp --port 443 --cidr 0.0.0.0/0

# Add port 8080
aws ec2 authorize-security-group-ingress --group-id sg-0abc123 --protocol tcp --port 8080 --cidr 0.0.0.0/0
```

### 9.8 What We Built — Visual Summary

```
                    INTERNET
                       |
                       |
              [Internet Gateway: my-igw]
                       |
                       |
    ┌──────────────────────────────────────┐
    │         VPC: my-custom-vpc           │
    │         CIDR: 10.0.0.0/16            │
    │                                      │
    │  ┌─────────────┐  ┌──────────────┐   │
    │  │ Public       │  │ Private      │   │
    │  │ Subnet       │  │ Subnet       │   │
    │  │ 10.0.1.0/24  │  │ 10.0.2.0/24  │   │
    │  │ AZ: a        │  │ AZ: b        │   │
    │  │              │  │              │   │
    │  │ Route Table:  │  │ Route Table: │   │
    │  │ 0.0.0.0/0    │  │ local only   │   │
    │  │ → IGW        │  │              │   │
    │  │              │  │              │   │
    │  │ [EC2 here    │  │ [DB here     │   │
    │  │  later]      │  │  later]      │   │
    │  └─────────────┘  └──────────────┘   │
    │                                      │
    │  Security Group: web-server-sg       │
    │  - Port 22 (SSH from your IP)        │
    │  - Port 80 (HTTP from anywhere)      │
    │  - Port 443 (HTTPS from anywhere)    │
    │  - Port 8080 (App from anywhere)     │
    └──────────────────────────────────────┘
```

---

## 10. Cleanup

### Delete Resources in This Order (dependencies matter!):

**Console Method:**

1. **Security Groups:** VPC → Security Groups → Select `web-server-sg` → Actions → Delete
2. **Subnets:** VPC → Subnets → Select each → Actions → Delete subnet
3. **Route Tables:** VPC → Route Tables → Select `my-public-rt` → Actions → Delete route table
4. **Internet Gateway:** VPC → Internet Gateways → Select `my-igw` → Actions → Detach from VPC → then Delete
5. **VPC:** VPC → Your VPCs → Select `my-custom-vpc` → Actions → Delete VPC

**CLI Method (use your actual IDs):**
```powershell
# Delete security group
aws ec2 delete-security-group --group-id sg-0abc123

# Delete subnets
aws ec2 delete-subnet --subnet-id subnet-public-id
aws ec2 delete-subnet --subnet-id subnet-private-id

# Disassociate and delete route table
aws ec2 delete-route-table --route-table-id rtb-0abc123

# Detach and delete internet gateway
aws ec2 detach-internet-gateway --internet-gateway-id igw-0abc123 --vpc-id vpc-0abc123
aws ec2 delete-internet-gateway --internet-gateway-id igw-0abc123

# Delete VPC
aws ec2 delete-vpc --vpc-id vpc-0abc123
```

---

## ✅ Part 3 Checklist

- [ ] Understand IP addresses (public vs private)
- [ ] Understand CIDR notation
- [ ] Understand common ports
- [ ] Know what a VPC is and why it matters
- [ ] Understand public vs private subnets
- [ ] Know what an Internet Gateway does
- [ ] Understand route tables
- [ ] Master Security Groups (stateful firewall)
- [ ] Know the difference between Security Groups and NACLs
- [ ] Understand Elastic IP costs
- [ ] Built a custom VPC with public + private subnets
- [ ] Created a security group with proper rules
- [ ] Cleaned up all resources

---

> 📖 **Next: Part 4 — EC2 (Elastic Compute Cloud)** — Launch your first virtual server, SSH into it, install software, and deploy an application!
