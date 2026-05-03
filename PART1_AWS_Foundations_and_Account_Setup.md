# 🚀 AWS Full Tutorial — Part 1: Foundations & Account Setup

> **Who is this for?** Beginners with a FREE TIER AWS account on a Windows laptop.
> **Goal:** Learn every concept, every click, every command — zero cost.

---

## Table of Contents — Part 1

1. [What is Cloud Computing?](#1-what-is-cloud-computing)
2. [AWS Global Infrastructure](#2-aws-global-infrastructure)
3. [AWS Free Tier — What You Get for Free](#3-aws-free-tier--what-you-get-for-free)
4. [Securing Your AWS Account (IAM)](#4-securing-your-aws-account-iam)
5. [Setting Up Billing Alerts (Never Get Charged!)](#5-setting-up-billing-alerts)
6. [Installing AWS CLI on Windows](#6-installing-aws-cli-on-windows)

---

## 1. What is Cloud Computing?

### 1.1 The Simple Explanation

Imagine you want to run a website. Traditionally, you would:
- Buy a physical server (₹50,000+)
- Set it up in your home/office
- Pay for electricity, internet, cooling
- If the server breaks, your website goes down

**Cloud Computing** means: Instead of buying your own server, you **rent** servers from companies like Amazon (AWS), Google (GCP), or Microsoft (Azure). You pay only for what you use — like electricity billing.

### 1.2 Types of Cloud Computing

| Type | What It Means | Example |
|------|--------------|---------|
| **IaaS** (Infrastructure as a Service) | You rent raw servers, storage, networking. YOU manage the OS, apps, everything else. | AWS EC2 (Virtual Machines) |
| **PaaS** (Platform as a Service) | You get a platform to deploy your code. THEY manage the OS, runtime, servers. | AWS Elastic Beanstalk |
| **SaaS** (Software as a Service) | You just use the software. THEY manage everything. | Gmail, Dropbox |

### 1.3 Cloud Deployment Models

| Model | Meaning |
|-------|---------|
| **Public Cloud** | Resources shared across many customers (AWS, Azure, GCP) |
| **Private Cloud** | Dedicated infrastructure for one organization |
| **Hybrid Cloud** | Mix of public + private cloud |

### 1.4 Key Benefits of Cloud

1. **Pay-as-you-go** — No upfront cost. Pay only for what you use
2. **Scalability** — Need more power? Add it in seconds. Need less? Remove it
3. **High Availability** — Your app runs in multiple locations, so if one fails, others take over
4. **Security** — AWS spends billions on security infrastructure
5. **Global Reach** — Deploy your app in 30+ countries in minutes

---

## 2. AWS Global Infrastructure

### 2.1 Regions

A **Region** is a physical location in the world where AWS has data centers.

**Examples:**
| Region Name | Region Code | Location |
|-------------|------------|----------|
| US East (N. Virginia) | `us-east-1` | Virginia, USA |
| Asia Pacific (Mumbai) | `ap-south-1` | Mumbai, India |
| Europe (London) | `eu-west-2` | London, UK |
| Asia Pacific (Singapore) | `ap-southeast-1` | Singapore |

**How to choose a Region:**
- **Closest to your users** → Lower latency (faster response)
- **Pricing** → Some regions are cheaper (us-east-1 is usually cheapest)
- **Compliance** → Some data must stay in certain countries
- **Service availability** → Not all services are in all regions

> 💡 **For this tutorial, we'll use `ap-south-1` (Mumbai) if you're in India, or `us-east-1` (N. Virginia) which has the most services and lowest prices.**

### 2.2 Availability Zones (AZs)

Each Region has **multiple Availability Zones** (usually 2-6).

An **Availability Zone** = One or more physical data centers with:
- Independent power supply
- Independent networking
- Independent cooling

**Why multiple AZs?**
If one data center catches fire or has a power outage, your app keeps running in another AZ.

```
Region: ap-south-1 (Mumbai)
├── AZ: ap-south-1a  (Data Center 1)
├── AZ: ap-south-1b  (Data Center 2)
└── AZ: ap-south-1c  (Data Center 3)
```

**Key Point:** AZs within a Region are connected with high-speed, low-latency networking.

### 2.3 Edge Locations

**Edge Locations** are mini data centers used for **caching content closer to users**.

- Used by **CloudFront** (AWS's CDN — Content Delivery Network)
- 400+ Edge Locations worldwide
- If someone in Japan visits your website hosted in Mumbai, CloudFront caches a copy in Tokyo's edge location, so it loads faster

### 2.4 Visual Summary

```
AWS Global Infrastructure
│
├── Region (ap-south-1 — Mumbai)
│   ├── Availability Zone A (ap-south-1a)
│   │   ├── Data Center 1
│   │   └── Data Center 2
│   ├── Availability Zone B (ap-south-1b)
│   │   ├── Data Center 3
│   │   └── Data Center 4
│   └── Availability Zone C (ap-south-1c)
│       └── Data Center 5
│
├── Region (us-east-1 — N. Virginia)
│   ├── AZ a, AZ b, AZ c, AZ d, AZ e, AZ f
│
└── 400+ Edge Locations worldwide
```

---

## 3. AWS Free Tier — What You Get for Free

### 3.1 Three Types of Free Tier

| Type | Duration | Example |
|------|----------|---------|
| **12 Months Free** | First 12 months after account creation | EC2, S3, RDS |
| **Always Free** | Never expires | Lambda, DynamoDB (limited) |
| **Trials** | Short-term free trials of specific services | SageMaker, Inspector |

### 3.2 Key Free Tier Limits (What We'll Use)

| Service | Free Tier Limit | What It Is |
|---------|----------------|------------|
| **EC2** | 750 hours/month of t2.micro (or t3.micro) | Virtual server |
| **S3** | 5 GB storage, 20,000 GET requests, 2,000 PUT requests | File/object storage |
| **Lambda** | 1 million requests/month, 400,000 GB-seconds | Serverless functions |
| **DynamoDB** | 25 GB storage, 25 read/write capacity units | NoSQL database |
| **RDS** | 750 hours/month of db.t2.micro or db.t3.micro | Relational database |
| **CloudWatch** | 10 custom metrics, 10 alarms | Monitoring |
| **SNS** | 1 million publishes | Notifications |
| **SQS** | 1 million requests | Message queue |
| **API Gateway** | 1 million API calls/month (12 months) | API management |
| **CloudFront** | 1 TB data transfer out | CDN |

### 3.3 ⚠️ DANGER ZONES — Things That Can Cost Money

> **CRITICAL: Read this carefully!**

| Danger | Why It Costs Money | How to Avoid |
|--------|--------------------|--------------|
| **Leaving EC2 running** | 750 hrs = ~31 days x 24 hrs. 2 instances = only ~15 days each | Stop/terminate instances when not using |
| **Elastic IPs not attached** | Unattached Elastic IPs cost $0.005/hour | Always release EIPs you don't need |
| **NAT Gateway** | NOT free tier. Costs ~$0.045/hour + data | Don't create NAT Gateways |
| **EBS Snapshots** | 30 GB free, beyond that costs money | Delete old snapshots |
| **Data Transfer** | First 1 GB/month is free outbound | Keep transfers minimal |
| **RDS Multi-AZ** | Only single-AZ is free tier eligible | Never enable Multi-AZ in free tier |
| **Forgetting to delete resources** | Resources keep running = keep charging | Always clean up after practice |

---

## 4. Securing Your AWS Account (IAM)

### 4.1 What is IAM?

**IAM = Identity and Access Management**

It controls **WHO** can access **WHAT** in your AWS account.

Think of it like a building's security system:
- **Root User** = Building owner (has ALL keys to everything)
- **IAM User** = Employee with a key card (has limited access)
- **IAM Group** = Department (all members get same access)
- **IAM Policy** = The rules written on the key card (what doors it opens)
- **IAM Role** = A temporary badge (like a visitor pass for AWS services)

### 4.2 Step-by-Step: Secure Your Root Account

#### Step 1: Enable MFA on Root Account

**Why?** The root account has unlimited power. If someone gets your password, they can rack up thousands in charges. MFA adds a second layer of security.

**What is MFA?** Multi-Factor Authentication. Even if someone knows your password, they need your phone to log in.

**Steps:**

1. Go to [https://console.aws.amazon.com](https://console.aws.amazon.com)
2. Sign in with your **Root user** email and password
3. Click on your **account name** (top-right corner of the screen)
4. Click **"Security credentials"**
5. Scroll down to **"Multi-factor authentication (MFA)"**
6. Click **"Assign MFA device"**
7. Give it a name like `my-phone-mfa`
8. Select **"Authenticator app"** → Click **Next**
9. Install one of these apps on your phone:
   - **Google Authenticator** (recommended)
   - **Microsoft Authenticator**
   - **Authy**
10. Click **"Show QR code"**
11. Open the authenticator app → Tap **"+"** → **Scan QR code**
12. Enter **two consecutive MFA codes** from the app (wait for code to change for the second one)
13. Click **"Add MFA"**

✅ **Done!** Your root account is now protected with MFA.

#### Step 2: Create an IAM Admin User (NEVER use root for daily work)

**Why?** Root user can do ANYTHING including closing the account. Create an admin IAM user for daily work.

**Steps:**

1. In the AWS Console, search for **"IAM"** in the search bar → Click **IAM**
2. In the left sidebar, click **"Users"**
3. Click **"Create user"** (blue button, top-right)
4. **User name:** `admin-user`
5. Check ✅ **"Provide user access to the AWS Management Console"**
6. Select **"I want to create an IAM user"**
7. Choose **"Custom password"** → Enter a strong password
8. Uncheck "User must create a new password at next sign-in" (optional)
9. Click **"Next"**

**Set Permissions:**

10. Select **"Attach policies directly"**
11. In the search bar, type `AdministratorAccess`
12. Check ✅ the box next to **"AdministratorAccess"**
13. Click **"Next"**
14. Review the settings → Click **"Create user"**

**Save Your Sign-in URL:**

15. You'll see a **Console sign-in URL** like:
    ```
    https://123456789012.signin.aws.amazon.com/console
    ```
16. **SAVE THIS URL** — Bookmark it!
17. Also save/download the password

**Enable MFA for This User Too:**

18. Click on the user name `admin-user`
19. Go to **"Security credentials"** tab
20. Under **"Multi-factor authentication (MFA)"** → Click **"Assign MFA device"**
21. Repeat the same MFA steps as before

✅ **From now on, always sign in as `admin-user`, NOT as root.**

### 4.3 Understanding IAM Policies (JSON)

IAM policies are written in JSON. Here's an example:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::my-bucket/*"
        }
    ]
}
```

**Breaking it down:**
| Field | Meaning |
|-------|---------|
| `Version` | Always `"2012-10-17"` (policy language version) |
| `Effect` | `"Allow"` or `"Deny"` |
| `Action` | What action is allowed/denied (e.g., `s3:GetObject` = read files) |
| `Resource` | Which specific resource (ARN = Amazon Resource Name) |

**Common Actions:**
| Action | Meaning |
|--------|---------|
| `s3:GetObject` | Download/read a file from S3 |
| `s3:PutObject` | Upload a file to S3 |
| `ec2:RunInstances` | Launch EC2 instances |
| `ec2:TerminateInstances` | Delete EC2 instances |
| `*` | ALL actions (dangerous!) |

### 4.4 Creating an IAM Group

**Why Groups?** Instead of attaching policies to each user, attach to a group.

**Steps:**

1. Go to **IAM** → **User groups** (left sidebar)
2. Click **"Create group"**
3. **Group name:** `developers`
4. Under "Attach permissions policies", search and check:
   - ✅ `AmazonS3FullAccess`
   - ✅ `AmazonEC2FullAccess`
5. Click **"Create user group"**

### 4.5 IAM Roles (For Services, Not Humans)

**What is a Role?**
A Role is like a temporary pass for AWS services. Example: You want your EC2 server to read files from S3. You create a Role that allows S3 access and attach it to the EC2 instance.

**We'll create roles later when we set up EC2 and Lambda.**

### 4.6 IAM Best Practices Summary

| Practice | Why |
|----------|-----|
| Never use root for daily work | Root can't be restricted |
| Enable MFA on all accounts | Prevents unauthorized access |
| Use groups to assign permissions | Easier to manage |
| Follow least privilege principle | Give only the permissions needed |
| Rotate access keys regularly | Reduces risk if keys are leaked |
| Use roles for AWS services | More secure than embedding keys |

---

## 5. Setting Up Billing Alerts

### 5.1 Why This Matters

If you accidentally leave a resource running, AWS will charge your credit card. Billing alerts warn you BEFORE charges get high.

### 5.2 Step-by-Step: Create a Billing Alarm

#### Step A: Enable Billing Alerts

1. Sign in as **root user** (billing settings require root)
2. Click your **account name** (top-right) → **"Account"**
3. In the left sidebar, scroll to **"Billing preferences"**
4. Under **"Invoice delivery preferences"** → Check ✅ **"PDF invoices delivered by email"**
5. Under **"Alert preferences"**:
   - Check ✅ **"Receive AWS Free Tier alerts"**
   - Enter your **email address**
   - Check ✅ **"Receive CloudWatch billing alerts"**
6. Click **"Save preferences"**

#### Step B: Create a Zero-Spend Budget

1. Search "Budgets" in the search bar → Click **"AWS Budgets"**
2. Click **"Create a budget"**
3. Select **"Use a template (simplified)"**
4. Choose **"Zero spend budget"**
5. **Budget name:** `my-zero-spend-budget`
6. **Email recipients:** Enter your email
7. Click **"Create budget"**

✅ You'll now get an email alert if AWS charges you even $0.01!

#### Step C: Create a CloudWatch Billing Alarm

1. Switch to **`us-east-1` (N. Virginia)** region (billing metrics only exist here)
2. Search for **"CloudWatch"** → Click it
3. Left sidebar → **"Alarms"** → **"All alarms"**
4. Click **"Create alarm"**
5. Click **"Select metric"**
6. Click **"Billing"** → **"Total Estimated Charge"**
7. Check ✅ **"EstimatedCharges"** (Currency = USD) → Click **"Select metric"**
8. Conditions:
   - **Threshold type:** Static
   - **Whenever EstimatedCharges is:** Greater than
   - **than:** `1` (alert if charges exceed $1)
9. Click **"Next"**
10. **Create new SNS topic:**
    - **Topic name:** `billing-alarm-topic`
    - **Email:** Enter your email
11. Click **"Create topic"** → Click **"Next"**
12. **Alarm name:** `billing-alarm-1-dollar`
13. Click **"Next"** → Click **"Create alarm"**
14. **CHECK YOUR EMAIL** → Click **"Confirm subscription"** in the SNS email

✅ You'll now get emailed if your bill exceeds $1!

---

## 6. Installing AWS CLI on Windows

### 6.1 What is AWS CLI?

**AWS CLI (Command Line Interface)** lets you manage AWS services from your terminal instead of clicking through the console.

### 6.2 Installation Steps

#### Step 1: Download and Install

1. Go to: `https://awscli.amazonaws.com/AWSCLIV2.msi`
2. Double-click `AWSCLIV2.msi`
3. Click **Next** → Accept license → **Next** → **Install** → **Finish**

#### Step 2: Verify Installation

```powershell
aws --version
```

**Expected Output:**
```
aws-cli/2.x.x Python/3.x.x Windows/10 exe/AMD64
```

### 6.3 Create Access Keys for CLI

1. Sign in as `admin-user`
2. Click your **account name** (top-right) → **"Security credentials"**
3. Scroll to **"Access keys"**
4. Click **"Create access key"**
5. Select **"Command Line Interface (CLI)"**
6. Check ✅ confirmation → Click **"Next"**
7. **Description:** `windows-laptop-cli`
8. Click **"Create access key"**
9. **⚠️ Download the .csv file or copy both keys NOW!** You won't see the Secret Key again!

### 6.4 Configure AWS CLI

```powershell
aws configure
```

```
AWS Access Key ID [None]: PASTE_YOUR_ACCESS_KEY_HERE
AWS Secret Access Key [None]: PASTE_YOUR_SECRET_KEY_HERE
Default region name [None]: ap-south-1
Default output format [None]: json
```

#### Verify Configuration

```powershell
aws sts get-caller-identity
```

**Expected Output:**
```json
{
    "UserId": "AIDAEXAMPLEID",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/admin-user"
}
```

### 6.5 Where Are Credentials Stored?

```
C:\Users\YOUR_USERNAME\.aws\credentials    ← Access keys
C:\Users\YOUR_USERNAME\.aws\config         ← Region and output format
```

> ⚠️ **NEVER share these files or commit them to Git!**

---

## ✅ Part 1 Checklist

- [ ] Understand cloud computing concepts (IaaS, PaaS, SaaS)
- [ ] Understand Regions, AZs, Edge Locations
- [ ] Know your Free Tier limits
- [ ] MFA enabled on root account
- [ ] IAM admin user created (`admin-user`)
- [ ] MFA enabled on admin user
- [ ] Zero-spend budget created
- [ ] CloudWatch billing alarm created
- [ ] AWS CLI installed and configured on Windows

---

> 📖 **Next: Part 2 — S3 (Simple Storage Service)** — Create buckets, upload files, host a static website, and set up access policies!
