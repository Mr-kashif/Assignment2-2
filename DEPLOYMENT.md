# 🚀 Deploy MERN Social Media App to AWS EC2 with Docker

This guide provides step-by-step instructions to deploy your Social Media Web App on an AWS EC2 instance using Docker and Docker Compose.

---

## 📋 Prerequisites


- AWS Account
- GitHub Account
- Basic knowledge of Linux commands
- MongoDB Atlas account (for database)

---

## 🔧 Part 1: Local Setup & Push to GitHub

### 1. Initialize Git Repository (if not already done)

```bash
cd E:\Socialmediaapp_Assignment
git init
git add .
git commit -m "Initial commit: MERN Social Media App with Docker support"
```

### 2. Create GitHub Repository

1. Go to [GitHub](https://github.com) and create a new repository
2. Name it: `Socialmediaapp_Assignment` (or your preferred name)
3. Don't initialize with README (we already have files)

### 3. Push to GitHub

```bash
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git
git branch -M main
git push -u origin main
```

**Important:** Make sure your `.env` file is NOT pushed to GitHub (add to `.gitignore`)

---

## 🖥️ Part 2: Setup AWS EC2 Instance

### 1. Launch EC2 Instance

1. Log in to [AWS Console](https://console.aws.amazon.com)
2. Go to **EC2 Dashboard**
3. Click **Launch Instance**
4. Configure:
   - **Name:** `social-media-app`
   - **AMI:** Ubuntu Server 22.04 LTS (Free tier eligible)
   - **Instance Type:** t2.medium (RECOMMENDED for build process)
   - **Key Pair:** Create new or use existing (download `.pem` file)
   - **Security Group:** Create with these rules:
     - **SSH (22)** - Your IP only (for security)
     - **HTTP (80)** - 0.0.0.0/0 (Allow all - for friends access)
     - **HTTPS (443)** - 0.0.0.0/0 (for future SSL)
     - ~~Custom TCP (4000)~~ - **DON'T open** (nginx will proxy)

5. **Storage:** 20 GB minimum (General Purpose SSD)
6. Click **Launch Instance**

### 2. Connect to EC2 Instance

#### Windows (PowerShell):
```powershell
ssh -i "path\to\your-key.pem" ubuntu@your-ec2-public-ip
```

#### Linux/Mac:
```bash
chmod 400 your-key.pem
ssh -i "your-key.pem" ubuntu@your-ec2-public-ip
```

---

## 🐳 Part 3: Install Docker & Docker Compose on EC2

### 1. Update System
```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Install Docker
```bash
# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add current user to docker group
sudo usermod -aG docker $USER

# Start Docker service
sudo systemctl enable docker
sudo systemctl start docker

# Verify installation
docker --version
```

### 3. Install Docker Compose
```bash
# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Make it executable
sudo chmod +x /usr/local/bin/docker-compose

# Verify installation
docker-compose --version
```

### 4. Log out and log back in
```bash
exit
# Then SSH again to apply docker group changes
ssh -i "your-key.pem" ubuntu@your-ec2-public-ip
```

---

## 📦 Part 4: Clone & Deploy Application

### 1. Install Git (if not installed)
```bash
sudo apt install git -y
```

### 2. Clone Your Repository
```bash
cd ~
git clone https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git
cd YOUR_REPO_NAME
```

### 3. Create Environment File
```bash
cd Server
nano .env
```

Paste your environment variables:
```env
MONGO_DB=mongodb+srv://your-username:your-password@your-cluster.mongodb.net/your-database?retryWrites=true&w=majority
PORT=4000
JWT_KEY=your-secret-jwt-key-here
```

Press `Ctrl + X`, then `Y`, then `Enter` to save.

### 4. Go back to root directory
```bash
cd ..
```

---

## 🚀 Part 5: Build & Run with Docker Compose

### 1. Build Docker Images
```bash
docker-compose build
```

This will take 5-10 minutes. It will:
- Build the Node.js backend
- Build the React frontend
- Setup nginx for serving the app

### 2. Start the Application
```bash
docker-compose up -d
```

The `-d` flag runs containers in detached mode (background).

### 3. Verify Containers are Running
```bash
docker-compose ps
```

You should see:
- `social-media-server` - Running on port 4000
- `social-media-client` - Running on port 80

### 4. Check Logs (if needed)
```bash
# All logs
docker-compose logs

# Server logs only
docker-compose logs server

# Client logs only
docker-compose logs client

# Follow logs in real-time
docker-compose logs -f
```

---

## 🌐 Part 6: Access Your Application

### 1. Get EC2 Public IP
```bash
curl http://checkip.amazonaws.com
```

### 2. Open in Browser
```
http://YOUR_EC2_PUBLIC_IP
```

Your Social Media App should now be live! 🎉

### 3. API Endpoint
```
http://YOUR_EC2_PUBLIC_IP/api
```

---

## 🔄 Part 7: Update Application (After Code Changes)

### 1. SSH into EC2
```bash
ssh -i "your-key.pem" ubuntu@your-ec2-public-ip
```

### 2. Navigate to Project
```bash
cd ~/YOUR_REPO_NAME
```

### 3. Pull Latest Changes
```bash
git pull origin main
```

### 4. Rebuild and Restart
```bash
# Stop containers
docker-compose down

# Rebuild images
docker-compose build

# Start containers
docker-compose up -d
```

---

## 🛠️ Useful Docker Commands

### Container Management
```bash
# Stop all containers
docker-compose down

# Start containers
docker-compose up -d

# Restart specific service
docker-compose restart server
docker-compose restart client

# View running containers
docker ps

# View all containers
docker ps -a
```

### Logs & Debugging
```bash
# View logs
docker-compose logs

# Follow logs
docker-compose logs -f server

# Execute command in container
docker exec -it social-media-server sh
```

### Cleanup
```bash
# Remove stopped containers
docker-compose down

# Remove containers and volumes
docker-compose down -v

# Remove all unused images
docker image prune -a

# Clean everything (careful!)
docker system prune -a
```

---

## 🔒 Part 8: Security Best Practices

### 1. Setup Firewall (UFW)
```bash
# Enable UFW
sudo ufw enable

# Allow SSH
sudo ufw allow 22/tcp

# Allow HTTP
sudo ufw allow 80/tcp

# Allow HTTPS (for future SSL)
sudo ufw allow 443/tcp

# Check status
sudo ufw status
```

### 2. Close Port 4000 (Backend should only be accessed via nginx)
Update Security Group in AWS Console to remove port 4000 access from 0.0.0.0/0

### 3. Setup SSL Certificate (Optional but Recommended)

#### Install Certbot
```bash
sudo apt install certbot python3-certbot-nginx -y
```

#### Get SSL Certificate (requires domain name)
```bash
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

---

## 📊 Part 9: Monitoring

### Check Application Status
```bash
# Container status
docker-compose ps

# Resource usage
docker stats

# Disk usage
docker system df
```

### Check System Resources
```bash
# Memory usage
free -h

# Disk usage
df -h

# CPU usage
top
```

---

## 🐛 Troubleshooting

### Container Won't Start
```bash
# Check logs
docker-compose logs

# Check specific service
docker-compose logs server
```

### MongoDB Connection Issues
1. Verify MongoDB Atlas IP whitelist includes EC2 IP
2. Check `.env` file has correct connection string
3. Test connection:
```bash
docker exec -it social-media-server sh
node -e "console.log(process.env.MONGO_DB)"
```

### Port Already in Use
```bash
# Check what's using port 80
sudo lsof -i :80

# Kill the process (replace PID)
sudo kill -9 PID
```

### Out of Memory
- Upgrade EC2 instance type
- Add swap space:
```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

---

## 📝 Additional Notes

### Environment Variables
Never commit `.env` files to Git. Always use `.env.example` as a template.

### MongoDB Atlas Setup
1. Create cluster
2. Create database user
3. Whitelist EC2 IP address (or 0.0.0.0/0 for any IP)
4. Get connection string

### Backup Strategy
```bash
# Backup uploaded images
docker cp social-media-server:/app/public ./backup/

# Automated backup script (optional)
# Create a cron job for regular backups
```

---

## 🎯 Quick Reference

### Start Everything
```bash
cd ~/YOUR_REPO_NAME
docker-compose up -d
```

### Stop Everything
```bash
docker-compose down
```

### Update & Deploy
```bash
git pull origin main
docker-compose down
docker-compose build
docker-compose up -d
```

---

## 📞 Support

If you encounter issues:
1. Check container logs: `docker-compose logs`
2. Verify EC2 security groups
3. Ensure MongoDB Atlas is accessible
4. Check `.env` file configuration

---

## ✅ Deployment Checklist

- [ ] GitHub repository created and code pushed
- [ ] EC2 instance launched with proper security groups
- [ ] Docker and Docker Compose installed
- [ ] Repository cloned to EC2
- [ ] `.env` file created with correct values
- [ ] Docker images built successfully
- [ ] Containers running
- [ ] Application accessible via browser
- [ ] MongoDB connection working
- [ ] Firewall configured
- [ ] (Optional) SSL certificate installed

---

**Congratulations! Your MERN Social Media App is now live on AWS EC2! 🎉**
