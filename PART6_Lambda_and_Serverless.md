# ⚡ AWS Full Tutorial — Part 6: Lambda & Serverless Computing

> **What is Serverless?** You write code. AWS runs it. You don't manage ANY servers. You pay only when your code runs (and Lambda's free tier gives you 1 million requests/month FREE).

---

## Table of Contents — Part 6

1. [Serverless Concepts](#1-serverless-concepts)
2. [AWS Lambda — Core Concepts](#2-aws-lambda--core-concepts)
3. [Creating Your First Lambda Function](#3-creating-your-first-lambda-function)
4. [Lambda with API Gateway (Build a REST API)](#4-lambda-with-api-gateway-build-a-rest-api)
5. [Lambda with S3 (Auto-Process Uploaded Files)](#5-lambda-with-s3-auto-process-uploaded-files)
6. [Lambda with DynamoDB](#6-lambda-with-dynamodb)
7. [Lambda Environment Variables & Layers](#7-lambda-environment-variables--layers)
8. [Cleanup](#8-cleanup)

---

## 1. Serverless Concepts

### 1.1 Traditional vs Serverless

| Feature | Traditional (EC2) | Serverless (Lambda) |
|---------|-------------------|---------------------|
| **Server management** | You manage everything | AWS manages everything |
| **Scaling** | You configure scaling | Automatic |
| **Billing** | Pay per hour (even idle) | Pay per request |
| **Always running?** | Yes (unless you stop it) | No — runs only when triggered |
| **Use case** | Full applications, databases | Event-driven tasks, APIs, automation |

### 1.2 When to Use Lambda vs EC2

| Use Lambda For | Use EC2 For |
|---------------|-------------|
| API endpoints | Full web applications |
| File processing (S3 uploads) | Databases |
| Scheduled tasks (cron jobs) | Long-running processes |
| Event-driven workflows | Apps needing > 15 min runtime |
| Lightweight microservices | Stateful applications |

### 1.3 Lambda Free Tier (Always Free!)

| What | Limit |
|------|-------|
| Requests | 1,000,000 per month |
| Compute time | 400,000 GB-seconds per month |
| This means | ~3.2 million seconds of execution at 128 MB |

> 💡 Lambda's free tier **never expires** — it's always free, not just the first 12 months!

---

## 2. AWS Lambda — Core Concepts

### 2.1 How Lambda Works

```
Event (trigger) → Lambda Function → Response
```

**Events can come from:**
- API Gateway (HTTP request)
- S3 (file uploaded)
- DynamoDB (data changed)
- CloudWatch Events (scheduled timer)
- SNS/SQS (message received)
- Manual invocation

### 2.2 Lambda Function Structure

Every Lambda function has:
- **Handler** — The function that AWS calls when triggered
- **Event** — Input data (what triggered the function)
- **Context** — Information about the invocation (time remaining, function name, etc.)
- **Response** — What the function returns

### 2.3 Supported Runtimes

| Runtime | Language |
|---------|----------|
| Node.js 18.x / 20.x | JavaScript |
| Python 3.11 / 3.12 | Python |
| Java 17 / 21 | Java |
| .NET 6 / 8 | C# |
| Go 1.x | Go |
| Ruby 3.2 / 3.3 | Ruby |
| Custom Runtime | Any language |

### 2.4 Lambda Limits

| Limit | Value |
|-------|-------|
| **Max execution time** | 15 minutes |
| **Memory** | 128 MB to 10,240 MB |
| **Package size (zipped)** | 50 MB |
| **Package size (unzipped)** | 250 MB |
| **Environment variables** | 4 KB total |
| **Concurrent executions** | 1,000 (default, can increase) |

---

## 3. Creating Your First Lambda Function

### 3.1 Console Method (Recommended for Learning)

1. Search for **"Lambda"** → Click **Lambda**
2. Click **"Create function"**
3. Select **"Author from scratch"**
4. **Function name:** `my-first-lambda`
5. **Runtime:** `Python 3.12`
6. **Architecture:** x86_64
7. **Permissions:** Keep default (creates a new role with basic Lambda permissions)
8. Click **"Create function"**

### 3.2 Write Your Lambda Code

You'll see a code editor in the browser. Replace the default code with:

```python
import json
import datetime

def lambda_handler(event, context):
    """
    This is the main handler function.
    AWS calls this function when the Lambda is triggered.
    
    Parameters:
    - event: The input data (what triggered the function)
    - context: Runtime info (function name, time remaining, etc.)
    """
    
    # Get current time
    current_time = datetime.datetime.now().isoformat()
    
    # Get the name from the event (if provided)
    name = "World"
    if event.get("queryStringParameters"):
        name = event["queryStringParameters"].get("name", "World")
    elif event.get("name"):
        name = event["name"]
    
    # Build response
    response = {
        "statusCode": 200,
        "headers": {
            "Content-Type": "application/json"
        },
        "body": json.dumps({
            "message": f"Hello, {name}! This is from AWS Lambda!",
            "timestamp": current_time,
            "function_name": context.function_name,
            "memory_limit_mb": context.memory_limit_in_mb,
            "remaining_time_ms": context.get_remaining_time_in_millis()
        })
    }
    
    return response
```

9. Click **"Deploy"** (saves the code)

### 3.3 Test Your Lambda

1. Click **"Test"** (next to Deploy)
2. **Configure test event:**
   - **Event name:** `test-event`
   - **Template:** hello-world
   - Replace the JSON with:
   ```json
   {
       "name": "AWS Student"
   }
   ```
3. Click **"Save"** then **"Test"**

**You'll see the output:**
```json
{
    "statusCode": 200,
    "headers": {"Content-Type": "application/json"},
    "body": "{\"message\": \"Hello, AWS Student! This is from AWS Lambda!\", ...}"
}
```

✅ Your first Lambda function works!

### 3.4 Lambda via CLI

```powershell
# Invoke the function
aws lambda invoke --function-name my-first-lambda --payload '{"name":"CLI User"}' --cli-binary-format raw-in-base64-out output.json

# View the response
cat output.json
```

---

## 4. Lambda with API Gateway (Build a REST API)

### 4.1 What is API Gateway?

**API Gateway** is a service that lets you create HTTP APIs. It acts as a "front door" that receives HTTP requests and triggers your Lambda function.

```
User → HTTP Request → API Gateway → Lambda Function → Response → User
```

**Free Tier:** 1 million API calls per month (for 12 months).

### 4.2 Create an API Gateway

1. Go to your Lambda function page
2. Click **"Add trigger"** (in the Function overview diagram)
3. Select **"API Gateway"** from the dropdown
4. **API type:** HTTP API (simpler and cheaper than REST API)
5. **Security:** Open (no auth for now — this is for learning)
6. Click **"Add"**

### 4.3 Get Your API URL

1. After adding, you'll see the API Gateway trigger in the diagram
2. Click on it — you'll see the **API endpoint** URL like:
   ```
   https://abc123xyz.execute-api.ap-south-1.amazonaws.com/default/my-first-lambda
   ```
3. **Open this URL in your browser!**

You'll see:
```json
{
    "message": "Hello, World! This is from AWS Lambda!",
    "timestamp": "2026-05-03T12:00:00",
    ...
}
```

### 4.4 Add Query Parameters

Try adding `?name=YourName` to the URL:
```
https://abc123xyz.execute-api.ap-south-1.amazonaws.com/default/my-first-lambda?name=Rahul
```

Response:
```json
{
    "message": "Hello, Rahul! This is from AWS Lambda!",
    ...
}
```

🎉 **You just built a serverless API!** No EC2, no servers, no maintenance.

### 4.5 Build a More Complete API

Create a new Lambda function for a TODO API:

1. Create new function: `todo-api`
2. Runtime: Python 3.12
3. Code:

```python
import json

# In-memory storage (resets when Lambda cold starts)
todos = []

def lambda_handler(event, context):
    http_method = event.get("requestContext", {}).get("http", {}).get("method", "GET")
    path = event.get("rawPath", "/")
    
    if http_method == "GET" and "/todos" in path:
        return {
            "statusCode": 200,
            "headers": {"Content-Type": "application/json"},
            "body": json.dumps({"todos": todos, "count": len(todos)})
        }
    
    elif http_method == "POST" and "/todos" in path:
        body = json.loads(event.get("body", "{}"))
        todo = {
            "id": len(todos) + 1,
            "task": body.get("task", "Untitled"),
            "done": False
        }
        todos.append(todo)
        return {
            "statusCode": 201,
            "headers": {"Content-Type": "application/json"},
            "body": json.dumps({"message": "Todo created!", "todo": todo})
        }
    
    else:
        return {
            "statusCode": 200,
            "headers": {"Content-Type": "application/json"},
            "body": json.dumps({
                "message": "TODO API",
                "endpoints": {
                    "GET /todos": "List all todos",
                    "POST /todos": "Create a todo (body: {task: '...'})"
                }
            })
        }
```

---

## 5. Lambda with S3 (Auto-Process Uploaded Files)

### 5.1 The Concept

When someone uploads a file to S3, Lambda automatically runs and processes it.

```
File uploaded to S3 → S3 sends event → Lambda processes the file
```

### 5.2 Create the Lambda Function

1. Create function: `s3-file-processor`
2. Runtime: Python 3.12
3. Code:

```python
import json
import boto3

s3_client = boto3.client('s3')

def lambda_handler(event, context):
    # Get info about the uploaded file from the event
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key = record['s3']['object']['key']
        size = record['s3']['object']['size']
        
        print(f"New file uploaded!")
        print(f"  Bucket: {bucket}")
        print(f"  File: {key}")
        print(f"  Size: {size} bytes")
        
        # Example: Get the file content (for text files)
        if key.endswith('.txt'):
            response = s3_client.get_object(Bucket=bucket, Key=key)
            content = response['Body'].read().decode('utf-8')
            print(f"  Content: {content[:200]}")  # First 200 chars
    
    return {
        'statusCode': 200,
        'body': json.dumps('File processed successfully!')
    }
```

4. Click **Deploy**

### 5.3 Add S3 Permissions to Lambda Role

1. Go to your Lambda function → **Configuration** tab → **Permissions**
2. Click on the **Role name** (opens IAM)
3. Click **"Add permissions"** → **"Attach policies"**
4. Search and add: `AmazonS3ReadOnlyAccess`
5. Click **"Add permissions"**

### 5.4 Add S3 Trigger

1. Go back to your Lambda function
2. Click **"Add trigger"**
3. Select **S3**
4. **Bucket:** Select your bucket (e.g., `my-first-bucket-rahul-2026`)
5. **Event types:** All object create events (s3:ObjectCreated:*)
6. **Prefix:** (leave empty — trigger for all files)
7. **Suffix:** `.txt` (only trigger for .txt files)
8. Check ✅ acknowledgment
9. Click **"Add"**

### 5.5 Test It!

Upload a text file to your S3 bucket:

```powershell
echo "This is a test file for Lambda processing!" > C:\Users\ASUS\Desktop\test-lambda.txt
aws s3 cp C:\Users\ASUS\Desktop\test-lambda.txt s3://my-first-bucket-rahul-2026/
```

Then check Lambda logs:
1. Lambda → `s3-file-processor` → **Monitor** tab → **View CloudWatch Logs**
2. Click the latest log stream
3. You'll see the print statements showing the file was processed!

---

## 6. Lambda with DynamoDB

### 6.1 What is DynamoDB?

**DynamoDB** is AWS's serverless NoSQL database. No servers to manage, automatically scales.

**Free Tier (Always Free):**
- 25 GB of storage
- 25 read capacity units
- 25 write capacity units

### 6.2 Create a DynamoDB Table

**Console:**

1. Search **"DynamoDB"** → Click it
2. Click **"Create table"**
3. **Table name:** `visitors`
4. **Partition key:** `visitorId` (String)
5. **Table settings:** Default settings
6. Click **"Create table"**

### 6.3 Lambda Function with DynamoDB

Create a new Lambda function: `visitor-counter`

```python
import json
import boto3
import uuid
from datetime import datetime

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('visitors')

def lambda_handler(event, context):
    # Record a new visitor
    visitor_id = str(uuid.uuid4())
    
    table.put_item(Item={
        'visitorId': visitor_id,
        'timestamp': datetime.now().isoformat(),
        'userAgent': event.get('headers', {}).get('user-agent', 'unknown'),
        'sourceIp': event.get('requestContext', {}).get('http', {}).get('sourceIp', 'unknown')
    })
    
    # Count total visitors
    response = table.scan(Select='COUNT')
    total_visitors = response['Count']
    
    return {
        'statusCode': 200,
        'headers': {
            'Content-Type': 'application/json',
            'Access-Control-Allow-Origin': '*'
        },
        'body': json.dumps({
            'message': 'Visit recorded!',
            'visitorId': visitor_id,
            'totalVisitors': total_visitors
        })
    }
```

**Add DynamoDB permissions to the Lambda role:**
1. Lambda → Configuration → Permissions → Click role name
2. Add policy: `AmazonDynamoDBFullAccess`

**Add API Gateway trigger** (same as before).

Now every time someone hits the API URL, it records a visit in DynamoDB!

---

## 7. Lambda Environment Variables & Layers

### 7.1 Environment Variables

Store configuration outside your code (database names, API keys, etc.).

1. Lambda → Configuration → **Environment variables**
2. Click **"Edit"**
3. Add variables:
   - Key: `TABLE_NAME`, Value: `visitors`
   - Key: `ENVIRONMENT`, Value: `production`
4. Click **"Save"**

**Access in code:**
```python
import os

table_name = os.environ.get('TABLE_NAME', 'default-table')
environment = os.environ.get('ENVIRONMENT', 'development')
```

### 7.2 Lambda Layers

Layers let you share code/libraries across multiple Lambda functions.

**Use case:** If 5 Lambda functions all need the `requests` library, create one layer with it instead of bundling it in each function.

For now, just know that layers exist — you'll use them in advanced projects.

---

## 8. Cleanup

**Delete Lambda Functions:**
1. Lambda → Functions → Select each → Actions → Delete

**Delete API Gateways:**
1. API Gateway → APIs → Select → Actions → Delete

**Delete DynamoDB Table:**
1. DynamoDB → Tables → Select `visitors` → Delete
2. Type "delete" to confirm

**CLI:**
```powershell
# Delete Lambda functions
aws lambda delete-function --function-name my-first-lambda
aws lambda delete-function --function-name todo-api
aws lambda delete-function --function-name s3-file-processor
aws lambda delete-function --function-name visitor-counter

# Delete DynamoDB table
aws dynamodb delete-table --table-name visitors
```

---

## ✅ Part 6 Checklist

- [ ] Understand serverless vs traditional computing
- [ ] Know Lambda's free tier limits
- [ ] Created a Lambda function in the console
- [ ] Tested Lambda with test events
- [ ] Connected Lambda to API Gateway (built a REST API)
- [ ] Connected Lambda to S3 (auto-process file uploads)
- [ ] Created a DynamoDB table
- [ ] Built Lambda + DynamoDB visitor counter
- [ ] Understand environment variables
- [ ] Cleaned up all resources

---

> 📖 **Next: Part 7 — Full Industry-Level Project** — Bring everything together into a real-world application!
