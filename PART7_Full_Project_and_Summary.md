# 🏗️ AWS Full Tutorial — Part 7: Full Industry-Level Project

> **Project:** Build a Serverless URL Shortener using S3 (frontend), Lambda + API Gateway (backend), and DynamoDB (database) — 100% free tier!

---

## Architecture

```
User's Browser
    |
    | (Static Website)
    v
S3 Static Website Hosting (Frontend - HTML/CSS/JS)
    |
    | API calls
    v
API Gateway (HTTP API)
    |
    | Triggers
    v
Lambda Functions (Python)
    |
    | Read/Write
    v
DynamoDB (NoSQL Database)
```

---

## Step 1: Create DynamoDB Table

1. Go to **DynamoDB** → **Create table**
2. **Table name:** `url-shortener`
3. **Partition key:** `shortId` (String)
4. **Settings:** Default settings
5. Click **Create table**

**CLI:**
```powershell
aws dynamodb create-table --table-name url-shortener --attribute-definitions AttributeName=shortId,AttributeType=S --key-schema AttributeName=shortId,KeyType=HASH --billing-mode PAY_PER_REQUEST
```

---

## Step 2: Create Lambda Function — Create Short URL

1. Go to **Lambda** → **Create function**
2. **Name:** `url-shortener-create`
3. **Runtime:** Python 3.12
4. Click **Create function**

**Code:**

```python
import json
import boto3
import string
import random
from datetime import datetime

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('url-shortener')

def generate_short_id(length=6):
    chars = string.ascii_letters + string.digits
    return ''.join(random.choices(chars, k=length))

def lambda_handler(event, context):
    # Handle CORS preflight
    if event.get('requestContext', {}).get('http', {}).get('method') == 'OPTIONS':
        return {'statusCode': 200, 'headers': {'Access-Control-Allow-Origin': '*', 'Access-Control-Allow-Methods': 'POST,GET,OPTIONS', 'Access-Control-Allow-Headers': 'Content-Type'}, 'body': ''}

    try:
        body = json.loads(event.get('body', '{}'))
        long_url = body.get('url', '')

        if not long_url:
            return {'statusCode': 400, 'headers': {'Access-Control-Allow-Origin': '*'}, 'body': json.dumps({'error': 'URL is required'})}

        if not long_url.startswith(('http://', 'https://')):
            long_url = 'https://' + long_url

        short_id = generate_short_id()

        table.put_item(Item={
            'shortId': short_id,
            'longUrl': long_url,
            'createdAt': datetime.now().isoformat(),
            'clicks': 0
        })

        return {
            'statusCode': 201,
            'headers': {'Content-Type': 'application/json', 'Access-Control-Allow-Origin': '*'},
            'body': json.dumps({'shortId': short_id, 'longUrl': long_url, 'message': 'URL shortened successfully!'})
        }
    except Exception as e:
        return {'statusCode': 500, 'headers': {'Access-Control-Allow-Origin': '*'}, 'body': json.dumps({'error': str(e)})}
```

5. Click **Deploy**

**Add DynamoDB permissions:**
- Configuration → Permissions → Click role name → Add `AmazonDynamoDBFullAccess`

---

## Step 3: Create Lambda Function — Redirect

1. Create function: `url-shortener-redirect`
2. Runtime: Python 3.12

```python
import json
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('url-shortener')

def lambda_handler(event, context):
    short_id = event.get('pathParameters', {}).get('shortId', '')

    if not short_id:
        return {'statusCode': 400, 'body': json.dumps({'error': 'Short ID required'})}

    try:
        response = table.get_item(Key={'shortId': short_id})
        item = response.get('Item')

        if not item:
            return {'statusCode': 404, 'headers': {'Content-Type': 'text/html'}, 'body': '<h1>404 - URL Not Found</h1>'}

        # Increment click counter
        table.update_item(
            Key={'shortId': short_id},
            UpdateExpression='SET clicks = clicks + :inc',
            ExpressionAttributeValues={':inc': 1}
        )

        return {
            'statusCode': 301,
            'headers': {'Location': item['longUrl'], 'Access-Control-Allow-Origin': '*'}
        }
    except Exception as e:
        return {'statusCode': 500, 'body': json.dumps({'error': str(e)})}
```

3. Deploy. Add same DynamoDB permissions to this function's role.

---

## Step 4: Create Lambda Function — List All URLs

1. Create function: `url-shortener-list`
2. Runtime: Python 3.12

```python
import json
import boto3
from decimal import Decimal

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('url-shortener')

class DecimalEncoder(json.JSONEncoder):
    def default(self, o):
        if isinstance(o, Decimal):
            return int(o)
        return super().default(o)

def lambda_handler(event, context):
    try:
        response = table.scan()
        items = response.get('Items', [])
        items.sort(key=lambda x: x.get('createdAt', ''), reverse=True)

        return {
            'statusCode': 200,
            'headers': {'Content-Type': 'application/json', 'Access-Control-Allow-Origin': '*'},
            'body': json.dumps({'urls': items, 'count': len(items)}, cls=DecimalEncoder)
        }
    except Exception as e:
        return {'statusCode': 500, 'headers': {'Access-Control-Allow-Origin': '*'}, 'body': json.dumps({'error': str(e)})}
```

3. Deploy. Add DynamoDB permissions.

---

## Step 5: Create API Gateway

1. Go to **API Gateway** → **Create API** → **HTTP API** → **Build**
2. **API name:** `url-shortener-api`
3. Click **Next** (skip integrations for now)
4. Click **Next** → **Next** → **Create**

**Add Routes & Integrations:**

5. Go to your API → **Routes** → **Create**
   - `POST /shorten` → Integrate with `url-shortener-create`
   - `GET /urls` → Integrate with `url-shortener-list`
   - `GET /{shortId}` → Integrate with `url-shortener-redirect`

**For each route:**
1. Click the route → **Attach integration** → **Create and attach integration**
2. **Integration type:** Lambda function
3. Select the corresponding function
4. Click **Create**

**Configure CORS:**
1. Go to **CORS** in the left sidebar
2. **Allowed origins:** `*`
3. **Allowed methods:** `GET, POST, OPTIONS`
4. **Allowed headers:** `Content-Type`
5. Click **Save**

**Get your API URL:** It'll be like `https://abc123.execute-api.ap-south-1.amazonaws.com`

**Test with PowerShell:**
```powershell
# Create a short URL
Invoke-RestMethod -Method POST -Uri "https://YOUR_API_URL/shorten" -ContentType "application/json" -Body '{"url":"https://github.com"}'

# List all URLs
Invoke-RestMethod -Uri "https://YOUR_API_URL/urls"
```

---

## Step 6: Create the Frontend (S3 Static Website)

### Create the S3 Bucket

1. S3 → Create bucket: `url-shortener-frontend-YOUR-NAME-2026`
2. Uncheck "Block all public access" → Acknowledge
3. Create bucket
4. Enable Static website hosting (Properties → Static website hosting → Enable → index: `index.html`, error: `index.html`)
5. Add public read bucket policy (Permissions tab):

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicRead",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::url-shortener-frontend-YOUR-NAME-2026/*"
        }
    ]
}
```

### Create index.html

Save this to `C:\Users\ASUS\Desktop\url-shortener-frontend\index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AWS URL Shortener</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700&display=swap');
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: 'Inter', sans-serif;
            background: linear-gradient(135deg, #0a0a1a 0%, #1a0a2e 50%, #0a1628 100%);
            color: #e0e0e0; min-height: 100vh;
        }
        .container { max-width: 800px; margin: 0 auto; padding: 40px 20px; }
        h1 { text-align: center; font-size: 2.5em; margin-bottom: 10px;
             background: linear-gradient(90deg, #a78bfa, #ec4899, #f59e0b);
             -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
        .subtitle { text-align: center; color: #888; margin-bottom: 40px; font-size: 1.1em; }
        .card {
            background: rgba(255,255,255,0.05); border-radius: 16px; padding: 30px;
            border: 1px solid rgba(255,255,255,0.1); margin-bottom: 30px;
            backdrop-filter: blur(10px);
        }
        .input-group { display: flex; gap: 10px; margin-bottom: 15px; }
        input[type="url"], input[type="text"] {
            flex: 1; padding: 14px 18px; border-radius: 10px;
            border: 1px solid rgba(255,255,255,0.15); background: rgba(255,255,255,0.08);
            color: white; font-size: 1em; outline: none; transition: border 0.3s;
        }
        input:focus { border-color: #a78bfa; }
        button {
            padding: 14px 28px; border-radius: 10px; border: none; cursor: pointer;
            font-size: 1em; font-weight: 600; transition: all 0.3s;
            background: linear-gradient(135deg, #a78bfa, #ec4899); color: white;
        }
        button:hover { transform: translateY(-2px); box-shadow: 0 5px 20px rgba(167,139,250,0.4); }
        .result { padding: 15px; border-radius: 10px; margin-top: 15px; display: none;
                  background: rgba(167,139,250,0.1); border: 1px solid rgba(167,139,250,0.3); }
        .result a { color: #a78bfa; text-decoration: none; font-weight: 600; word-break: break-all; }
        .result a:hover { text-decoration: underline; }
        table { width: 100%; border-collapse: collapse; margin-top: 15px; }
        th { text-align: left; padding: 12px; color: #a78bfa; border-bottom: 1px solid rgba(255,255,255,0.1); }
        td { padding: 12px; border-bottom: 1px solid rgba(255,255,255,0.05); }
        td a { color: #ec4899; text-decoration: none; }
        td a:hover { text-decoration: underline; }
        .clicks { color: #f59e0b; font-weight: 600; }
        .badge { display: inline-block; padding: 4px 10px; border-radius: 20px; font-size: 0.8em;
                 background: rgba(167,139,250,0.2); color: #a78bfa; }
        .loading { text-align: center; padding: 20px; color: #888; }
        .empty { text-align: center; padding: 30px; color: #666; }
        .stats { display: flex; gap: 20px; margin-bottom: 30px; }
        .stat-card { flex: 1; padding: 20px; border-radius: 12px; text-align: center;
                     background: rgba(255,255,255,0.03); border: 1px solid rgba(255,255,255,0.08); }
        .stat-number { font-size: 2em; font-weight: 700; color: #a78bfa; }
        .stat-label { color: #888; font-size: 0.9em; margin-top: 5px; }
    </style>
</head>
<body>
    <div class="container">
        <h1>⚡ URL Shortener</h1>
        <p class="subtitle">Built with AWS Lambda, API Gateway, DynamoDB & S3</p>

        <div class="stats">
            <div class="stat-card"><div class="stat-number" id="totalUrls">0</div><div class="stat-label">Total URLs</div></div>
            <div class="stat-card"><div class="stat-number" id="totalClicks">0</div><div class="stat-label">Total Clicks</div></div>
        </div>

        <div class="card">
            <h2 style="margin-bottom:15px;">🔗 Shorten a URL</h2>
            <div class="input-group">
                <input type="url" id="urlInput" placeholder="Enter your long URL here..." />
                <button onclick="shortenUrl()">Shorten</button>
            </div>
            <div class="result" id="result">
                <p>✅ Short URL: <a id="shortUrl" href="#" target="_blank"></a></p>
                <p style="margin-top:8px;color:#888;font-size:0.9em;">Original: <span id="origUrl"></span></p>
            </div>
        </div>

        <div class="card">
            <h2 style="margin-bottom:15px;">📋 All Shortened URLs</h2>
            <div id="urlList"><div class="loading">Loading...</div></div>
        </div>

        <p style="text-align:center;color:#555;margin-top:30px;">
            🛠️ Powered by AWS Free Tier | S3 + Lambda + API Gateway + DynamoDB
        </p>
    </div>

    <script>
        // ⚠️ REPLACE THIS with your actual API Gateway URL!
        const API_URL = 'https://YOUR_API_GATEWAY_URL_HERE';

        async function shortenUrl() {
            const urlInput = document.getElementById('urlInput');
            const url = urlInput.value.trim();
            if (!url) { alert('Please enter a URL'); return; }

            try {
                const res = await fetch(`${API_URL}/shorten`, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ url })
                });
                const data = await res.json();

                const shortUrl = `${API_URL}/${data.shortId}`;
                document.getElementById('shortUrl').href = shortUrl;
                document.getElementById('shortUrl').textContent = shortUrl;
                document.getElementById('origUrl').textContent = data.longUrl;
                document.getElementById('result').style.display = 'block';

                urlInput.value = '';
                loadUrls();
            } catch (err) {
                alert('Error: ' + err.message);
            }
        }

        async function loadUrls() {
            try {
                const res = await fetch(`${API_URL}/urls`);
                const data = await res.json();
                const urls = data.urls || [];
                const container = document.getElementById('urlList');

                document.getElementById('totalUrls').textContent = urls.length;
                document.getElementById('totalClicks').textContent = urls.reduce((sum, u) => sum + (u.clicks || 0), 0);

                if (urls.length === 0) {
                    container.innerHTML = '<div class="empty">No URLs shortened yet. Try creating one above!</div>';
                    return;
                }

                let html = '<table><tr><th>Short ID</th><th>Original URL</th><th>Clicks</th><th>Created</th></tr>';
                urls.forEach(u => {
                    const shortLink = `${API_URL}/${u.shortId}`;
                    const displayUrl = u.longUrl.length > 40 ? u.longUrl.substring(0, 40) + '...' : u.longUrl;
                    const date = new Date(u.createdAt).toLocaleDateString();
                    html += `<tr>
                        <td><a href="${shortLink}" target="_blank">${u.shortId}</a></td>
                        <td title="${u.longUrl}">${displayUrl}</td>
                        <td><span class="clicks">${u.clicks || 0}</span></td>
                        <td><span class="badge">${date}</span></td>
                    </tr>`;
                });
                html += '</table>';
                container.innerHTML = html;
            } catch (err) {
                document.getElementById('urlList').innerHTML = '<div class="empty">Failed to load URLs. Check API connection.</div>';
            }
        }

        document.getElementById('urlInput').addEventListener('keypress', (e) => { if (e.key === 'Enter') shortenUrl(); });
        loadUrls();
    </script>
</body>
</html>
```

### Upload to S3

```powershell
aws s3 cp C:\Users\ASUS\Desktop\url-shortener-frontend\index.html s3://url-shortener-frontend-YOUR-NAME-2026/index.html --content-type text/html
```

### Get Your Website URL

Go to S3 bucket → Properties → Static website hosting → Copy the endpoint URL.

**Open it in your browser** — your full-stack serverless app is live! 🎉

---

## Step 7: Test the Complete Application

1. Open your S3 website URL in the browser
2. Enter a long URL (e.g., `https://docs.aws.amazon.com/lambda/latest/dg/welcome.html`)
3. Click **Shorten** — you get a short URL
4. Click the short URL — it redirects to the original!
5. Check the click counter increases

---

## Step 8: Complete Cleanup

**⚠️ DELETE EVERYTHING to avoid charges:**

```powershell
# Delete Lambda functions
aws lambda delete-function --function-name url-shortener-create
aws lambda delete-function --function-name url-shortener-redirect
aws lambda delete-function --function-name url-shortener-list

# Delete DynamoDB table
aws dynamodb delete-table --table-name url-shortener

# Empty and delete S3 bucket
aws s3 rm s3://url-shortener-frontend-YOUR-NAME-2026/ --recursive
aws s3 rb s3://url-shortener-frontend-YOUR-NAME-2026

# Delete API Gateway (get API ID first)
aws apigatewayv2 get-apis
aws apigatewayv2 delete-api --api-id YOUR_API_ID

# Delete IAM roles created by Lambda (go to IAM → Roles → delete ones starting with url-shortener)
```

**Console cleanup:**
1. Lambda → Delete all functions
2. DynamoDB → Delete table
3. S3 → Empty bucket → Delete bucket
4. API Gateway → Delete API
5. IAM → Roles → Delete Lambda execution roles
6. CloudWatch → Log groups → Delete `/aws/lambda/url-shortener-*` log groups

---

## ✅ Part 7 Checklist

- [ ] Created DynamoDB table for URL storage
- [ ] Built 3 Lambda functions (create, redirect, list)
- [ ] Set up API Gateway with routes
- [ ] Created a beautiful frontend
- [ ] Hosted frontend on S3
- [ ] Tested the full application end-to-end
- [ ] Cleaned up ALL resources

---

## 🎓 What You've Learned in This Tutorial

| Part | Topic | Key Skills |
|------|-------|-----------|
| 1 | Foundations | Cloud concepts, IAM, MFA, Billing alerts, AWS CLI |
| 2 | S3 | Buckets, objects, policies, static website hosting |
| 3 | Networking | VPC, subnets, IGW, route tables, security groups |
| 4 | EC2 | Launch instances, SSH, Linux, deploy apps, EBS, AMIs |
| 5 | Docker | Containers, Dockerfile, Docker Compose, ECR |
| 6 | Serverless | Lambda, API Gateway, DynamoDB, event-driven architecture |
| 7 | Project | Full-stack serverless app combining all services |

---

## 🔥 Next Steps After This Tutorial

1. **Get AWS Certified** — Start with AWS Cloud Practitioner (CLF-C02)
2. **Learn Terraform** — Infrastructure as Code (define AWS resources in code)
3. **Learn CI/CD on AWS** — CodePipeline, CodeBuild, CodeDeploy
4. **Explore ECS/EKS** — Run Docker containers at scale
5. **Learn CloudFormation** — AWS's native IaC tool
6. **Study for Solutions Architect Associate** — The most popular AWS certification

> 🎉 **Congratulations!** You've completed the AWS Full Tutorial. You now have hands-on experience with the most important AWS services — all within the free tier!
