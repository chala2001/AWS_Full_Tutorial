# 🐳 AWS Full Tutorial — Part 5: Docker on EC2 & ECR

> **What is Docker?** Docker lets you package your application and ALL its dependencies into a container — a lightweight, portable box that runs the same way everywhere.

---

## Table of Contents — Part 5

1. [Docker Concepts](#1-docker-concepts)
2. [Docker on EC2 — Hands-On](#2-docker-on-ec2--hands-on)
3. [Creating a Dockerfile](#3-creating-a-dockerfile)
4. [Building and Running Docker Containers](#4-building-and-running-docker-containers)
5. [Docker Compose](#5-docker-compose)
6. [ECR — Elastic Container Registry](#6-ecr--elastic-container-registry)
7. [Full Project: Dockerized App on EC2](#7-full-project-dockerized-app-on-ec2)
8. [Cleanup](#8-cleanup)

---

## 1. Docker Concepts

### 1.1 Why Docker?

**Problem:** "It works on my machine!" — Your app works on your laptop but breaks on the server because of different OS, different Node.js version, missing libraries, etc.

**Solution:** Docker packages EVERYTHING your app needs into a container. The container runs identically everywhere.

### 1.2 Key Terminology

| Term | Meaning | Analogy |
|------|---------|---------|
| **Image** | A blueprint/template of your app + dependencies | A recipe |
| **Container** | A running instance of an image | A dish made from the recipe |
| **Dockerfile** | Instructions to build an image | The recipe written on paper |
| **Registry** | A storage place for images | A cookbook library |
| **Docker Hub** | Public registry (like GitHub for Docker images) | Public library |
| **ECR** | AWS's private registry | Your private bookshelf |

### 1.3 Docker vs Virtual Machines

| Feature | Docker Containers | Virtual Machines |
|---------|------------------|-----------------|
| **Size** | Megabytes | Gigabytes |
| **Startup** | Seconds | Minutes |
| **OS** | Shares host OS kernel | Runs full separate OS |
| **Isolation** | Process-level | Hardware-level |
| **Performance** | Near-native | Some overhead |

---

## 2. Docker on EC2 — Hands-On

### 2.1 Prerequisites

SSH into your EC2 instance and make sure Docker is installed:

```bash
# If Docker is not installed yet:
sudo yum install -y docker
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ec2-user

# Log out and back in
exit
```

SSH back in, then verify:
```bash
docker --version
docker ps
```

### 2.2 Your First Container

```bash
# Run a simple hello-world container
docker run hello-world
```

This:
1. Pulls the `hello-world` image from Docker Hub
2. Creates a container from it
3. Runs it (prints a message)
4. Container exits

### 2.3 Run an Nginx Web Server in Docker

```bash
# Run Nginx in background (-d = detach, -p = port mapping)
docker run -d --name my-nginx -p 80:80 nginx
```

**What this does:**
- `-d` = Run in background (detached mode)
- `--name my-nginx` = Name the container "my-nginx"
- `-p 80:80` = Map port 80 on EC2 to port 80 inside the container
- `nginx` = Use the official Nginx image

**Now open your browser:** `http://YOUR_EC2_PUBLIC_IP`

You should see the Nginx welcome page! 🎉

### 2.4 Essential Docker Commands

```bash
# List running containers
docker ps

# List ALL containers (including stopped)
docker ps -a

# Stop a container
docker stop my-nginx

# Start a stopped container
docker start my-nginx

# View container logs
docker logs my-nginx

# Follow logs in real-time
docker logs -f my-nginx

# Execute a command inside a running container
docker exec -it my-nginx bash

# Inside the container, you can:
ls /usr/share/nginx/html/
cat /usr/share/nginx/html/index.html
exit

# Remove a container (must be stopped first)
docker stop my-nginx
docker rm my-nginx

# List downloaded images
docker images

# Remove an image
docker rmi nginx

# Remove ALL stopped containers
docker container prune

# Remove ALL unused images
docker image prune
```

---

## 3. Creating a Dockerfile

### 3.1 What is a Dockerfile?

A Dockerfile is a text file with instructions to build a Docker image. Each instruction creates a layer.

### 3.2 Create a Node.js App with Dockerfile

```bash
# Create project directory
mkdir ~/docker-app && cd ~/docker-app

# Create the app
cat > app.js << 'EOF'
const express = require('express');
const os = require('os');
const app = express();

app.get('/', (req, res) => {
    res.send(`
    <html>
    <head><title>Dockerized App</title></head>
    <body style="font-family: Arial; background: linear-gradient(135deg, #0f0c29, #302b63, #24243e);
                 color: white; display: flex; justify-content: center; align-items: center;
                 height: 100vh; margin: 0;">
        <div style="text-align: center; padding: 40px; background: rgba(255,255,255,0.05);
                    border-radius: 20px; border: 1px solid rgba(255,255,255,0.1);">
            <h1>🐳 Dockerized on AWS EC2!</h1>
            <p>Container Hostname: ${os.hostname()}</p>
            <p>Node.js Version: ${process.version}</p>
            <p>Platform: ${os.platform()} ${os.arch()}</p>
            <p>Memory: ${Math.round(os.totalmem() / 1024 / 1024)} MB</p>
            <p>Server Time: ${new Date().toISOString()}</p>
        </div>
    </body>
    </html>
    `);
});

app.get('/api/health', (req, res) => {
    res.json({
        status: 'healthy',
        container: os.hostname(),
        uptime: process.uptime(),
        memory: process.memoryUsage()
    });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, '0.0.0.0', () => {
    console.log(`Server running on port ${PORT}`);
});
EOF

# Create package.json
cat > package.json << 'EOF'
{
    "name": "docker-app",
    "version": "1.0.0",
    "description": "Dockerized Node.js app on AWS EC2",
    "main": "app.js",
    "scripts": {
        "start": "node app.js"
    },
    "dependencies": {
        "express": "^4.18.2"
    }
}
EOF

# Create .dockerignore (like .gitignore for Docker)
cat > .dockerignore << 'EOF'
node_modules
npm-debug.log
.git
.gitignore
EOF
```

### 3.3 Write the Dockerfile

```bash
cat > Dockerfile << 'EOF'
# Step 1: Use Node.js base image
FROM node:18-alpine

# Step 2: Set working directory inside container
WORKDIR /app

# Step 3: Copy package files first (for better caching)
COPY package*.json ./

# Step 4: Install dependencies
RUN npm install --production

# Step 5: Copy app source code
COPY . .

# Step 6: Expose the port the app runs on
EXPOSE 3000

# Step 7: Define the command to run the app
CMD ["npm", "start"]
EOF
```

**Dockerfile Instructions Explained:**
| Instruction | Meaning |
|------------|---------|
| `FROM` | Base image to build on (node:18-alpine = small Linux with Node.js) |
| `WORKDIR` | Set the working directory inside the container |
| `COPY` | Copy files from your machine into the container |
| `RUN` | Execute a command during build (e.g., npm install) |
| `EXPOSE` | Document which port the app uses |
| `CMD` | The command to run when the container starts |

---

## 4. Building and Running Docker Containers

### 4.1 Build the Image

```bash
cd ~/docker-app

# Build the image (-t = tag/name the image)
docker build -t my-node-app:v1 .
```

**What happens:**
1. Docker reads the Dockerfile
2. Pulls the `node:18-alpine` base image
3. Creates each layer step by step
4. Tags the final image as `my-node-app:v1`

```bash
# Verify the image was created
docker images
```

### 4.2 Run the Container

```bash
# Run the container
docker run -d --name my-app -p 8080:3000 my-node-app:v1
```

**Explanation:**
- `-p 8080:3000` = Map EC2's port 8080 → container's port 3000
- External users hit port 8080, Docker forwards to container's 3000

**Visit:** `http://YOUR_EC2_PUBLIC_IP:8080`

### 4.3 Check It's Working

```bash
# View running containers
docker ps

# View logs
docker logs my-app

# Check health endpoint
curl http://localhost:8080/api/health
```

### 4.4 Update and Rebuild

```bash
# Stop and remove old container
docker stop my-app && docker rm my-app

# Edit your app.js if needed, then rebuild
docker build -t my-node-app:v2 .

# Run new version
docker run -d --name my-app -p 8080:3000 my-node-app:v2
```

---

## 5. Docker Compose

### 5.1 What is Docker Compose?

Docker Compose lets you define and run **multi-container applications**. Example: Your app + a database + a cache — all defined in one file.

### 5.2 Install Docker Compose on EC2

```bash
# Download Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Make it executable
sudo chmod +x /usr/local/bin/docker-compose

# Verify
docker-compose --version
```

### 5.3 Create a Multi-Container App

```bash
mkdir ~/compose-app && cd ~/compose-app

# Create docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8080:3000"
    environment:
      - NODE_ENV=production
      - REDIS_HOST=redis
    depends_on:
      - redis
    restart: always

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    restart: always
EOF
```

Copy your app files (app.js, package.json, Dockerfile, .dockerignore) into `~/compose-app/`.

```bash
cp ~/docker-app/* ~/compose-app/
```

### 5.4 Run with Docker Compose

```bash
# Build and start all services
docker-compose up -d --build

# View running services
docker-compose ps

# View logs
docker-compose logs

# Stop all services
docker-compose down

# Stop and remove volumes
docker-compose down -v
```

---

## 6. ECR — Elastic Container Registry

### 6.1 What is ECR?

**ECR (Elastic Container Registry)** is AWS's private Docker image registry. Like Docker Hub, but private and integrated with AWS.

**Free Tier:** 500 MB of storage per month (always free).

### 6.2 Create an ECR Repository

**Console:**

1. Search for **"ECR"** or **"Elastic Container Registry"**
2. Click **"Get Started"** or **"Create repository"**
3. **Visibility:** Private
4. **Repository name:** `my-node-app`
5. Leave other settings as default
6. Click **"Create repository"**

**CLI:**
```powershell
aws ecr create-repository --repository-name my-node-app --region ap-south-1
```

### 6.3 Push Your Docker Image to ECR

**Step 1: Authenticate Docker to ECR** (run from EC2 instance)

```bash
# Get your AWS account ID
AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
REGION="ap-south-1"

# Authenticate Docker to ECR
aws ecr get-login-password --region $REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com
```

> ⚠️ For this to work on EC2, your instance needs an IAM Role with ECR permissions, OR you need to configure AWS CLI on the EC2 instance.

**Attach an IAM Role to EC2 (if not done):**

1. Go to **IAM** → **Roles** → **Create role**
2. **Trusted entity:** AWS service → EC2
3. **Permissions:** Search and add `AmazonEC2ContainerRegistryFullAccess`
4. **Role name:** `EC2-ECR-Access`
5. Create the role
6. Go to **EC2** → Select your instance → **Actions** → **Security** → **Modify IAM role**
7. Select `EC2-ECR-Access` → **Update IAM role**

Now try the authentication again.

**Step 2: Tag your image for ECR**

```bash
# Tag the image
docker tag my-node-app:v1 $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/my-node-app:v1
docker tag my-node-app:v1 $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/my-node-app:latest
```

**Step 3: Push to ECR**

```bash
docker push $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/my-node-app:v1
docker push $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/my-node-app:latest
```

**Step 4: Verify in Console**

Go to ECR → `my-node-app` → You should see your image versions!

### 6.4 Pull Image from ECR

On any EC2 instance (or locally):
```bash
# Authenticate first
aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com

# Pull the image
docker pull $AWS_ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com/my-node-app:latest

# Run it
docker run -d -p 8080:3000 $AWS_ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com/my-node-app:latest
```

---

## 7. Full Project: Dockerized App on EC2

### Architecture

```
Your Laptop (Windows)
    |
    | Push code to GitHub
    v
GitHub Repository
    |
    | SSH into EC2, git pull
    v
EC2 Instance (Amazon Linux)
    |
    | docker build & run
    v
Docker Container (Node.js App on port 8080)
    |
    | Accessible via public IP
    v
Users access: http://EC2_PUBLIC_IP:8080
```

### Steps Summary

1. **On your Windows laptop:**
   - Create the app (app.js, package.json, Dockerfile)
   - Push to GitHub

2. **On EC2:**
   ```bash
   # Clone the repo
   git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git
   cd YOUR_REPO

   # Build Docker image
   docker build -t my-app:latest .

   # Run it
   docker run -d --name my-app -p 8080:3000 --restart always my-app:latest

   # Verify
   curl http://localhost:8080
   ```

3. **For updates:**
   ```bash
   cd YOUR_REPO
   git pull
   docker stop my-app && docker rm my-app
   docker build -t my-app:latest .
   docker run -d --name my-app -p 8080:3000 --restart always my-app:latest
   ```

---

## 8. Cleanup

```bash
# On EC2: Stop and remove all containers
docker stop $(docker ps -aq)
docker rm $(docker ps -aq)

# Remove all images
docker rmi $(docker images -q)

# Remove all volumes
docker volume prune -f
```

**Delete ECR Repository (Console):**
1. Go to ECR → Select repository → Delete

**CLI:**
```powershell
aws ecr delete-repository --repository-name my-node-app --force --region ap-south-1
```

**Don't forget to stop/terminate EC2 if done!**

---

## ✅ Part 5 Checklist

- [ ] Understand Docker concepts (images, containers, Dockerfile)
- [ ] Ran your first Docker container (hello-world, nginx)
- [ ] Wrote a Dockerfile for a Node.js app
- [ ] Built and ran a custom Docker image
- [ ] Understand Docker Compose for multi-container apps
- [ ] Created an ECR repository
- [ ] Pushed an image to ECR
- [ ] Pulled and ran an image from ECR
- [ ] Cleaned up all resources

---

> 📖 **Next: Part 6 — Lambda & Serverless Computing** — Run code without managing ANY servers!
