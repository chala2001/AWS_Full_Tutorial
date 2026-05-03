# 🪣 AWS Full Tutorial — Part 2: S3 (Simple Storage Service)

> **What is S3?** Amazon S3 is an **object storage service**. Think of it as an unlimited hard drive in the cloud where you can store any type of file (images, videos, backups, code, datasets).

---

## Table of Contents — Part 2

1. [S3 Core Concepts](#1-s3-core-concepts)
2. [Creating Your First S3 Bucket (Console)](#2-creating-your-first-s3-bucket-console)
3. [Uploading and Managing Objects (Console)](#3-uploading-and-managing-objects-console)
4. [S3 Using AWS CLI](#4-s3-using-aws-cli)
5. [S3 Bucket Policies and Access Control](#5-s3-bucket-policies-and-access-control)
6. [Hosting a Static Website on S3](#6-hosting-a-static-website-on-s3)
7. [S3 Storage Classes](#7-s3-storage-classes)
8. [S3 Versioning](#8-s3-versioning)
9. [S3 Lifecycle Rules](#9-s3-lifecycle-rules)
10. [Cleanup — Delete Everything](#10-cleanup--delete-everything)

---

## 1. S3 Core Concepts

### 1.1 Key Terminology

| Term | Meaning | Real-World Analogy |
|------|---------|-------------------|
| **Bucket** | A container that holds your files | A folder on your desktop |
| **Object** | Any file stored in S3 (image, video, text, etc.) | A file inside that folder |
| **Key** | The full path/name of an object inside a bucket | The file path like `photos/vacation/beach.jpg` |
| **Value** | The actual content/data of the file | The actual image data |
| **Region** | Where your bucket physically lives | Which city your storage locker is in |
| **ARN** | Amazon Resource Name — unique identifier | Like an address for any AWS resource |

### 1.2 S3 Bucket Rules

- Bucket names must be **globally unique** (no one else in the world can have the same name)
- Bucket names must be **3-63 characters** long
- Bucket names can only contain **lowercase letters, numbers, and hyphens**
- Bucket names must **start with a letter or number**
- Buckets are created in a **specific Region** (choose one close to you)

### 1.3 S3 Object Rules

- Maximum object size: **5 TB** (5,000 GB)
- If uploading more than **5 GB**, you must use **multi-part upload**
- Each object has **metadata** (data about the data — content type, last modified, etc.)

### 1.4 Free Tier Limits for S3

| What | Limit |
|------|-------|
| Storage | 5 GB |
| GET requests | 20,000 per month |
| PUT requests | 2,000 per month |
| Data transfer OUT | 100 GB per month (across all services) |

---

## 2. Creating Your First S3 Bucket (Console)

### Step-by-Step:

1. Sign in to AWS Console as `admin-user`
2. Make sure you're in your preferred region (e.g., `ap-south-1` Mumbai)
3. Search for **"S3"** in the search bar → Click **S3**
4. Click **"Create bucket"** (orange button)

**Configure the bucket:**

5. **Bucket type:** General purpose
6. **Bucket name:** `my-first-bucket-YOUR-NAME-2026`
   - Example: `my-first-bucket-rahul-2026`
   - ⚠️ Must be globally unique! Add your name + year
7. **AWS Region:** Select your preferred region (e.g., Asia Pacific (Mumbai) ap-south-1)
8. **Object Ownership:** ACLs disabled (recommended) ← Keep default
9. **Block Public Access settings for this bucket:**
   - Keep ✅ **"Block all public access"** checked (default — we'll change this later for static website hosting)
10. **Bucket Versioning:** Disable (we'll enable later)
11. **Default encryption:**
    - **Encryption type:** Server-side encryption with Amazon S3 managed keys (SSE-S3) ← Keep default
    - **Bucket Key:** Enable ← Keep default
12. Click **"Create bucket"**

✅ **Your first S3 bucket is created!**

---

## 3. Uploading and Managing Objects (Console)

### 3.1 Upload a File

1. Click on your bucket name (e.g., `my-first-bucket-rahul-2026`)
2. Click **"Upload"** (orange button)
3. Click **"Add files"**
4. Select any file from your computer (e.g., a text file, image, etc.)
5. Scroll down → Click **"Upload"**
6. Wait for the green "Succeeded" message → Click **"Close"**

✅ Your file is now in S3!

### 3.2 Create a Folder

S3 doesn't actually have "folders" — it's a flat structure. But the console lets you create folder-like prefixes.

1. Click **"Create folder"**
2. **Folder name:** `images`
3. Click **"Create folder"**
4. Click into the `images/` folder
5. Upload an image file here

### 3.3 Download a File

1. Click the checkbox next to your file
2. Click **"Download"**

### 3.4 View Object Details

1. Click on the file name (not checkbox)
2. You can see:
   - **Object URL** — the URL to access this file (won't work yet, bucket is private)
   - **ARN** — the unique identifier
   - **Size, type, last modified**
   - **Metadata** — content type, etc.

---

## 4. S3 Using AWS CLI

### 4.1 List All Buckets

```powershell
aws s3 ls
```

**Output:**
```
2026-05-03 12:30:00 my-first-bucket-rahul-2026
```

### 4.2 Create a Bucket via CLI

```powershell
aws s3 mb s3://my-cli-bucket-YOUR-NAME-2026 --region ap-south-1
```

**Output:**
```
make_bucket: my-cli-bucket-YOUR-NAME-2026
```

### 4.3 Upload a File

First, create a test file on your Windows machine:

```powershell
echo "Hello from AWS CLI!" > C:\Users\ASUS\Desktop\test-upload.txt
```

Upload it to S3:

```powershell
aws s3 cp C:\Users\ASUS\Desktop\test-upload.txt s3://my-first-bucket-rahul-2026/
```

**Output:**
```
upload: .\test-upload.txt to s3://my-first-bucket-rahul-2026/test-upload.txt
```

### 4.4 Upload an Entire Folder

```powershell
# Create a test folder with some files
mkdir C:\Users\ASUS\Desktop\my-website
echo "<h1>Hello World</h1>" > C:\Users\ASUS\Desktop\my-website\index.html
echo "body { color: blue; }" > C:\Users\ASUS\Desktop\my-website\style.css

# Upload entire folder
aws s3 sync C:\Users\ASUS\Desktop\my-website s3://my-first-bucket-rahul-2026/website/
```

### 4.5 List Objects in a Bucket

```powershell
# List all objects
aws s3 ls s3://my-first-bucket-rahul-2026/

# List objects in a "folder"
aws s3 ls s3://my-first-bucket-rahul-2026/website/

# List recursively (all objects including sub-folders)
aws s3 ls s3://my-first-bucket-rahul-2026/ --recursive
```

### 4.6 Download a File

```powershell
aws s3 cp s3://my-first-bucket-rahul-2026/test-upload.txt C:\Users\ASUS\Desktop\downloaded.txt
```

### 4.7 Download Entire Bucket

```powershell
aws s3 sync s3://my-first-bucket-rahul-2026/ C:\Users\ASUS\Desktop\s3-download\
```

### 4.8 Delete a File

```powershell
aws s3 rm s3://my-first-bucket-rahul-2026/test-upload.txt
```

### 4.9 Delete All Files in a Bucket

```powershell
aws s3 rm s3://my-first-bucket-rahul-2026/ --recursive
```

### 4.10 Delete a Bucket

```powershell
# Bucket must be empty first
aws s3 rb s3://my-cli-bucket-YOUR-NAME-2026
```

---

## 5. S3 Bucket Policies and Access Control

### 5.1 What are Bucket Policies?

Bucket policies are **JSON documents** that define who can do what with your bucket and its objects.

### 5.2 Access Control Methods

| Method | Use Case |
|--------|----------|
| **Bucket Policy** | Control access at the bucket level (most common) |
| **IAM Policy** | Control access for specific IAM users/groups/roles |
| **ACL (Access Control List)** | Legacy method — AWS recommends NOT using this |
| **Block Public Access** | Master switch to block all public access |

### 5.3 Example: Allow Public Read Access (for website hosting)

This policy allows anyone on the internet to READ files from your bucket:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::my-first-bucket-rahul-2026/*"
        }
    ]
}
```

**Breaking it down:**
| Field | Meaning |
|-------|---------|
| `Sid` | Statement ID — just a label/name |
| `Principal: "*"` | Who this applies to. `*` = everyone |
| `Action: "s3:GetObject"` | What they can do — only read/download |
| `Resource` | Which bucket + `/*` means all objects inside |

### 5.4 How to Apply a Bucket Policy (Console)

1. Go to your bucket → **"Permissions"** tab
2. Scroll to **"Bucket policy"** → Click **"Edit"**
3. Paste the JSON policy
4. Click **"Save changes"**

> ⚠️ If "Block Public Access" is ON, public policies won't work. You'll need to disable it first (next section).

---

## 6. Hosting a Static Website on S3

This is one of the coolest free things you can do with S3! You'll host an actual website accessible via a URL.

### 6.1 Prepare Your Website Files

Create these files on your Windows machine:

**`index.html`:**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My AWS Website</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: 'Segoe UI', Arial, sans-serif;
            background: linear-gradient(135deg, #0f0c29, #302b63, #24243e);
            color: white;
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
        }
        .container {
            text-align: center;
            padding: 40px;
            background: rgba(255,255,255,0.1);
            border-radius: 20px;
            backdrop-filter: blur(10px);
            border: 1px solid rgba(255,255,255,0.2);
            max-width: 600px;
        }
        h1 {
            font-size: 2.5em;
            margin-bottom: 20px;
            background: linear-gradient(90deg, #f093fb, #f5576c);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
        }
        p { font-size: 1.2em; line-height: 1.8; opacity: 0.9; }
        .badge {
            display: inline-block;
            margin-top: 20px;
            padding: 10px 25px;
            background: linear-gradient(90deg, #f093fb, #f5576c);
            border-radius: 25px;
            font-weight: bold;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🚀 Hello from AWS S3!</h1>
        <p>This website is hosted entirely on Amazon S3.<br>
           No servers. No EC2. Just S3 static hosting.</p>
        <div class="badge">Hosted on AWS Free Tier</div>
    </div>
</body>
</html>
```

**`error.html`:**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>404 - Page Not Found</title>
    <style>
        body {
            font-family: 'Segoe UI', Arial, sans-serif;
            background: #1a1a2e;
            color: white;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }
        .error-box { text-align: center; }
        h1 { font-size: 6em; color: #f5576c; }
        p { font-size: 1.5em; opacity: 0.7; }
    </style>
</head>
<body>
    <div class="error-box">
        <h1>404</h1>
        <p>Oops! Page not found.</p>
    </div>
</body>
</html>
```

Save both files to: `C:\Users\ASUS\Desktop\my-s3-website\`

### 6.2 Create a Bucket for Website Hosting

1. Go to **S3** → Click **"Create bucket"**
2. **Bucket name:** `my-website-YOUR-NAME-2026` (e.g., `my-website-rahul-2026`)
3. **Region:** Your preferred region
4. **Uncheck** ❌ **"Block all public access"**
   - A warning will appear — Check ✅ the acknowledgment box ("I acknowledge...")
5. Leave everything else as default
6. Click **"Create bucket"**

### 6.3 Upload Website Files

```powershell
aws s3 sync C:\Users\ASUS\Desktop\my-s3-website s3://my-website-rahul-2026/
```

Or upload via Console: Click bucket → Upload → Add both files.

### 6.4 Enable Static Website Hosting

1. Click on your website bucket
2. Go to **"Properties"** tab
3. Scroll ALL the way down to **"Static website hosting"**
4. Click **"Edit"**
5. **Static website hosting:** Enable
6. **Hosting type:** Host a static website
7. **Index document:** `index.html`
8. **Error document:** `error.html`
9. Click **"Save changes"**

### 6.5 Add Bucket Policy for Public Access

1. Go to **"Permissions"** tab
2. Scroll to **"Bucket policy"** → Click **"Edit"**
3. Paste this policy (replace `YOUR-BUCKET-NAME`):

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::my-website-rahul-2026/*"
        }
    ]
}
```

4. Click **"Save changes"**

### 6.6 Get Your Website URL

1. Go to **"Properties"** tab
2. Scroll to **"Static website hosting"**
3. You'll see a URL like:
   ```
   http://my-website-rahul-2026.s3-website.ap-south-1.amazonaws.com
   ```
4. **Click it!** Your website is LIVE on the internet! 🎉

---

## 7. S3 Storage Classes

S3 offers different storage classes with different pricing and availability:

| Storage Class | Use Case | Availability | Cost |
|---------------|----------|-------------|------|
| **S3 Standard** | Frequently accessed data | 99.99% | Highest |
| **S3 Intelligent-Tiering** | Unknown access patterns | 99.9% | Auto-optimizes |
| **S3 Standard-IA** | Infrequent access, but need quick retrieval | 99.9% | Lower storage, higher retrieval |
| **S3 One Zone-IA** | Infrequent access, single AZ (less resilient) | 99.5% | Even lower |
| **S3 Glacier Instant** | Archive, but need instant access | 99.9% | Very low |
| **S3 Glacier Flexible** | Archive, retrieval in minutes-hours | 99.99% | Very low |
| **S3 Glacier Deep Archive** | Long-term archive, retrieval in 12+ hours | 99.99% | Cheapest |

> 💡 **For free tier, everything is in S3 Standard. You don't need to worry about storage classes until you have production workloads.**

---

## 8. S3 Versioning

### 8.1 What is Versioning?

Versioning keeps **multiple versions** of the same file. If you accidentally overwrite or delete a file, you can recover the previous version.

### 8.2 Enable Versioning

**Console:**
1. Go to your bucket → **"Properties"** tab
2. Under **"Bucket Versioning"** → Click **"Edit"**
3. Select **"Enable"** → Click **"Save changes"**

**CLI:**
```powershell
aws s3api put-bucket-versioning --bucket my-first-bucket-rahul-2026 --versioning-configuration Status=Enabled
```

### 8.3 How Versioning Works

1. Upload `file.txt` with content "Version 1" → S3 assigns Version ID `v1`
2. Upload `file.txt` again with "Version 2" → S3 assigns Version ID `v2`
3. Both versions are stored! You can access either one.
4. Deleting `file.txt` just adds a **"delete marker"** — previous versions still exist.

### 8.4 List Object Versions (CLI)

```powershell
aws s3api list-object-versions --bucket my-first-bucket-rahul-2026
```

> ⚠️ **Versioning uses more storage** — each version counts toward your 5 GB free tier limit. Enable only when needed for practice, then disable.

---

## 9. S3 Lifecycle Rules

### 9.1 What are Lifecycle Rules?

Automatically move or delete objects based on age. Example: Move files older than 30 days to cheaper storage, delete files older than 365 days.

### 9.2 Create a Lifecycle Rule (Console)

1. Go to your bucket → **"Management"** tab
2. Under **"Lifecycle rules"** → Click **"Create lifecycle rule"**
3. **Rule name:** `move-to-glacier-after-30-days`
4. **Choose a rule scope:** Apply to all objects in the bucket
5. Check ✅ **"I acknowledge..."**
6. **Lifecycle rule actions:** Check ✅ **"Transition current versions of objects between storage classes"**
7. **Transition:**
   - **Storage class:** S3 Glacier Flexible Retrieval
   - **Days after object creation:** 30
8. Click **"Create rule"**

> 💡 For free tier practice, create the rule to learn, then delete it. Lifecycle rules themselves don't cost money — they just automate moving/deleting objects.

---

## 10. Cleanup — Delete Everything

### ⚠️ ALWAYS clean up after practicing to avoid charges!

#### Delete Bucket Contents + Bucket (CLI):

```powershell
# Delete all objects (including versioned objects)
aws s3 rm s3://my-first-bucket-rahul-2026/ --recursive

# If versioning was enabled, also delete all versions:
aws s3api delete-objects --bucket my-first-bucket-rahul-2026 --delete "$(aws s3api list-object-versions --bucket my-first-bucket-rahul-2026 --query '{Objects: Versions[].{Key:Key,VersionId:VersionId}}' --output json)"

# Delete the bucket
aws s3 rb s3://my-first-bucket-rahul-2026

# Repeat for other buckets
aws s3 rm s3://my-website-rahul-2026/ --recursive
aws s3 rb s3://my-website-rahul-2026
```

#### Delete via Console:

1. Go to **S3** → Check the bucket → Click **"Empty"**
2. Type "permanently delete" to confirm → Click **"Empty"**
3. Go back → Check the bucket → Click **"Delete"**
4. Type the bucket name to confirm → Click **"Delete bucket"**

---

## ✅ Part 2 Checklist

- [ ] Understand S3 concepts (buckets, objects, keys)
- [ ] Created an S3 bucket via Console
- [ ] Uploaded files via Console and CLI
- [ ] Used `aws s3 cp`, `sync`, `ls`, `rm` commands
- [ ] Understand bucket policies (JSON)
- [ ] Hosted a static website on S3
- [ ] Understand storage classes
- [ ] Practiced versioning
- [ ] Understand lifecycle rules
- [ ] Cleaned up all resources

---

> 📖 **Next: Part 3 — VPC, Subnets & Networking** — Understand virtual networks, subnets, route tables, internet gateways, and security groups before launching EC2!
