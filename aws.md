# Complete MERN Stack Deployment Guide - AWS EC2 + GoDaddy Domain

## Table of Contents

1. [AWS EC2 Instance Setup](#aws-ec2-instance-setup)
2. [Server Initial Configuration](#server-initial-configuration)
3. [Installing Required Software](#installing-required-software)
4. [Project Structure and Deployment](#project-structure-and-deployment)
5. [Environment Variables Configuration](#environment-variables-configuration)
6. [Nginx Configuration](#nginx-configuration)
7. [SSL Certificate Setup](#ssl-certificate-setup)
8. [GoDaddy Domain Configuration](#godaddy-domain-configuration)
9. [Process Management with PM2](#process-management-with-pm2)
10. [Security and Firewall](#security-and-firewall)
11. [Monitoring and Maintenance](#monitoring-and-maintenance)

---

## 1. AWS EC2 Instance Setup

### Step 1.1: Launch EC2 Instance

1. **Login to AWS Console**

   - Go to AWS Management Console
   - Navigate to EC2 Dashboard

2. **Launch Instance**

   ```
   Click "Launch Instance"
   ```

3. **Configure Instance**

   - **Name**: `mern-production-server`
   - **AMI**: Amazon Linux 2023 AMI (64-bit x86)
   - **Instance Type**: t3.medium (minimum for production)
   - **Key Pair**: Create new or select existing
   - **Security Group**: Create new with following rules:
     - SSH (22) - Your IP
     - HTTP (80) - Anywhere
     - HTTPS (443) - Anywhere
     - Custom TCP (3000) - Anywhere (for backend API)
     - Custom TCP (5000) - Anywhere (optional, for development)

4. **Storage Configuration**

   - Root Volume: 20-30 GB GP3
   - Add additional volume if needed

5. **Launch Instance**

### Step 1.2: Connect to Instance

**For Amazon Linux:**

```bash
# Connect via SSH
ssh -i "your-key-pair.pem" ec2-user@your-instance-public-ip
```

**For Ubuntu:**

```bash
# Connect via SSH
ssh -i "your-key-pair.pem" ubuntu@your-instance-public-ip
```

```bash
# Make key file secure (if needed)
chmod 400 your-key-pair.pem
```

---

## 2. Server Initial Configuration

### Step 2.1: Update System

**For Amazon Linux:**

```bash
# Update all packages
sudo dnf update -y

# Install essential packages
sudo dnf install -y wget curl git htop nano vim
```

**For Ubuntu:**

```bash
# Update package list
sudo apt update

# Upgrade all packages
sudo apt upgrade -y

# Install essential packages
sudo apt install -y wget curl git htop nano vim software-properties-common
```

### Step 2.2: Create Application Directory Structure

**For Amazon Linux:**

```bash
# Create main application directory
sudo mkdir -p /var/www
sudo chown ec2-user:ec2-user /var/www
cd /var/www

# Create project directories
mkdir -p mern-app/{frontend,backend,logs,scripts}
cd mern-app
```

**For Ubuntu:**

```bash
# Create main application directory
sudo mkdir -p /var/www
sudo chown ubuntu:ubuntu /var/www
cd /var/www

# Create project directories
mkdir -p mern-app/{frontend,backend,logs,scripts}
cd mern-app
```

**Recommended Directory Structure:**

```
/var/www/mern-app/
├── frontend/          # React build files
├── backend/           # Node.js backend
├── logs/              # Application logs
├── scripts/           # Deployment scripts
└── .env               # Environment variables
```

---

## 3. Installing Required Software

### Step 3.1: Install Node.js and npm

**For Amazon Linux:**

```bash
# Install Node.js 18.x (LTS)
curl -fsSL https://rpm.nodesource.com/setup_18.x | sudo bash -
sudo dnf install -y nodejs

# Verify installation
node --version
npm --version

# Install yarn (optional but recommended)
sudo npm install -g yarn
```

**For Ubuntu:**

```bash
# Install Node.js 18.x (LTS)
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Verify installation
node --version
npm --version

# Install yarn (optional but recommended)
sudo npm install -g yarn

# Alternative method using snap
# sudo snap install node --classic
```

### Step 3.2: Install Nginx

**For Amazon Linux:**

```bash
# Install Nginx
sudo dnf install -y nginx

# Start and enable Nginx
sudo systemctl start nginx
sudo systemctl enable nginx

# Check status
sudo systemctl status nginx
```

**For Ubuntu:**

```bash
# Install Nginx
sudo apt install -y nginx

# Start and enable Nginx
sudo systemctl start nginx
sudo systemctl enable nginx

# Check status
sudo systemctl status nginx

# Allow Nginx through UFW (if UFW is enabled)
sudo ufw allow 'Nginx Full'
```

### Step 3.3: Install PM2 (Process Manager)

```bash
# Install PM2 globally
sudo npm install -g pm2

# Setup PM2 startup
pm2 startup
# Follow the instructions provided by the command above
```

### Step 3.4: Install Git

**For Amazon Linux:**

```bash
# Git is usually pre-installed, but ensure it's there
sudo dnf install -y git

# Configure git (replace with your details)
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

**For Ubuntu:**

```bash
# Git is usually pre-installed, but ensure it's there
sudo apt install -y git

# Configure git (replace with your details)
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

---

## 4. Project Structure and Deployment

### Step 4.1: Clone Repository

```bash
# Navigate to project directory
cd /var/www/mern-app

# Clone your repository
git clone https://github.com/yourusername/your-mern-repo.git temp-repo

# Move files to proper structure
mv temp-repo/backend/* backend/
mv temp-repo/frontend/* frontend/
rm -rf temp-repo

# Alternative: Clone directly into subdirectories
git clone https://github.com/yourusername/your-backend-repo.git backend
git clone https://github.com/yourusername/your-frontend-repo.git frontend
```

### Step 4.2: Backend Setup

```bash
cd /var/www/mern-app/backend

# Install dependencies
npm install

# If you have package-lock.json, use:
npm ci --production
```

### Step 4.3: Frontend Setup and Build

```bash
cd /var/www/mern-app/frontend

# Install dependencies
npm install

# Build for production
npm run build

# The build folder will contain the production-ready files
ls -la build/
```

---

## 5. Environment Variables Configuration

### Step 5.1: Create Backend .env File

```bash
# Create environment file for backend
cd /var/www/mern-app/backend
nano .env
```

**Sample Backend .env File:**

```env
# Server Configuration
NODE_ENV=production
PORT=3000
HOST=0.0.0.0

# Database Configuration
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/database?retryWrites=true&w=majority
DB_NAME=your_database_name

# JWT Configuration
JWT_SECRET=your-super-secret-jwt-key-here
JWT_EXPIRE=7d

# CORS Configuration
CORS_ORIGIN=https://yourdomain.in,https://admin.yourdomain.in

# File Upload Configuration
UPLOAD_PATH=./uploads
MAX_FILE_SIZE=10485760

# Email Configuration (if using)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password

# Other API Keys
CLOUDINARY_CLOUD_NAME=your-cloud-name
CLOUDINARY_API_KEY=your-api-key
CLOUDINARY_API_SECRET=your-api-secret
```

### Step 5.2: Create Frontend Environment File

```bash
# Create environment file for frontend build
cd /var/www/mern-app/frontend
nano .env.production
```

**Sample Frontend .env.production File:**

```env
REACT_APP_API_URL=https://backend.yourdomain.in
REACT_APP_ADMIN_URL=https://admin.yourdomain.in
REACT_APP_ENVIRONMENT=production
REACT_APP_GOOGLE_ANALYTICS_ID=GA-XXXXXXXXX
```

### Step 5.3: Secure Environment Files

**For Amazon Linux:**

```bash
# Set proper permissions for .env files
chmod 600 /var/www/mern-app/backend/.env
chmod 600 /var/www/mern-app/frontend/.env.production

# Make sure only ec2-user can read these files
sudo chown ec2-user:ec2-user /var/www/mern-app/backend/.env
sudo chown ec2-user:ec2-user /var/www/mern-app/frontend/.env.production
```

**For Ubuntu:**

```bash
# Set proper permissions for .env files
chmod 600 /var/www/mern-app/backend/.env
chmod 600 /var/www/mern-app/frontend/.env.production

# Make sure only ubuntu user can read these files
sudo chown ubuntu:ubuntu /var/www/mern-app/backend/.env
sudo chown ubuntu:ubuntu /var/www/mern-app/frontend/.env.production
```

---

## 6. Nginx Configuration

### Step 6.1: Create Nginx Configuration

```bash
# Create Nginx configuration file
sudo nano /etc/nginx/conf.d/mern-app.conf
```

**Complete Nginx Configuration:**

```nginx
# Rate limiting
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
limit_req_zone $binary_remote_addr zone=login:10m rate=5r/m;

# Main domain - Frontend
server {
    listen 80;
    server_name yourdomain.in www.yourdomain.in;

    # Redirect to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name yourdomain.in www.yourdomain.in;

    # SSL Configuration (will be configured later)
    ssl_certificate /etc/letsencrypt/live/yourdomain.in/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.in/privkey.pem;

    # Security headers
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    # Root directory for frontend
    root /var/www/mern-app/frontend/build;
    index index.html;

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    # Handle React Router
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Static assets caching
    location /static/ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # API proxy to backend (if needed)
    location /api/ {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;

        # Rate limiting
        limit_req zone=api burst=20 nodelay;
    }
}

# Backend API subdomain
server {
    listen 80;
    server_name backend.yourdomain.in;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name backend.yourdomain.in;

    # SSL Configuration
    ssl_certificate /etc/letsencrypt/live/yourdomain.in/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.in/privkey.pem;

    # Security headers
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";

    # CORS headers
    add_header Access-Control-Allow-Origin "https://yourdomain.in, https://admin.yourdomain.in";
    add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS";
    add_header Access-Control-Allow-Headers "Origin, X-Requested-With, Content-Type, Accept, Authorization";

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;

        # Rate limiting
        limit_req zone=api burst=20 nodelay;
    }

    # Special rate limiting for login endpoints
    location ~* /(login|register|forgot-password) {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        limit_req zone=login burst=5 nodelay;
    }
}

# Admin subdomain
server {
    listen 80;
    server_name admin.yourdomain.in;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name admin.yourdomain.in;

    # SSL Configuration
    ssl_certificate /etc/letsencrypt/live/yourdomain.in/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.in/privkey.pem;

    # Security headers
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";

    # Admin panel location (if separate build)
    root /var/www/mern-app/frontend/build;
    index index.html;

    # If you have separate admin build, use:
    # root /var/www/mern-app/admin/build;

    location / {
        # Basic auth for admin (optional)
        # auth_basic "Admin Area";
        # auth_basic_user_file /etc/nginx/.htpasswd;

        try_files $uri $uri/ /index.html;
    }

    # Proxy API calls to backend
    location /api/ {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### Step 6.2: Test and Apply Nginx Configuration

```bash
# Test Nginx configuration
sudo nginx -t

# If test passes, reload Nginx
sudo systemctl reload nginx

# Check Nginx status
sudo systemctl status nginx
```

---

## 7. SSL Certificate Setup

### Step 7.1: Install Certbot

**For Amazon Linux:**

```bash
# Install Certbot
sudo dnf install -y python3-pip
sudo pip3 install certbot certbot-nginx
```

**For Ubuntu:**

```bash
# Install Certbot
sudo apt install -y certbot python3-certbot-nginx

# Alternative method using snap (recommended for Ubuntu)
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

### Step 7.2: Generate SSL Certificates

```bash
# Generate certificates for all domains
sudo certbot --nginx -d yourdomain.in -d www.yourdomain.in -d backend.yourdomain.in -d admin.yourdomain.in

# Follow the prompts:
# - Enter email address
# - Agree to terms
# - Choose whether to share email with EFF
# - Select redirect to HTTPS (option 2)
```

### Step 7.3: Setup Auto-renewal

```bash
# Test auto-renewal
sudo certbot renew --dry-run

# Setup cron job for auto-renewal
sudo crontab -e

# Add this line to run renewal check twice daily:
0 12 * * * /usr/local/bin/certbot renew --quiet
```

---

## 8. GoDaddy Domain Configuration

### Step 8.1: Get Your EC2 Public IP

```bash
# Get your instance's public IP
curl http://checkip.amazonaws.com
```

### Step 8.2: Configure DNS in GoDaddy

1. **Login to GoDaddy Account**

   - Go to GoDaddy DNS Manager
   - Select your domain

2. **Add/Modify DNS Records**

   **A Records:**

   ```
   Type: A
   Name: @
   Value: YOUR_EC2_PUBLIC_IP
   TTL: 1 Hour

   Type: A
   Name: www
   Value: YOUR_EC2_PUBLIC_IP
   TTL: 1 Hour

   Type: A
   Name: backend
   Value: YOUR_EC2_PUBLIC_IP
   TTL: 1 Hour

   Type: A
   Name: admin
   Value: YOUR_EC2_PUBLIC_IP
   TTL: 1 Hour
   ```

   **CNAME Records (Alternative approach):**

   ```
   Type: CNAME
   Name: www
   Value: yourdomain.in
   TTL: 1 Hour

   Type: CNAME
   Name: backend
   Value: yourdomain.in
   TTL: 1 Hour

   Type: CNAME
   Name: admin
   Value: yourdomain.in
   TTL: 1 Hour
   ```

3. **Save Changes**
   - DNS propagation can take 24-48 hours
   - Use `dig yourdomain.in` to check propagation

### Step 8.3: Verify DNS Configuration

```bash
# Check DNS resolution
nslookup yourdomain.in
nslookup www.yourdomain.in
nslookup backend.yourdomain.in
nslookup admin.yourdomain.in

# Or use dig
dig yourdomain.in
dig backend.yourdomain.in
dig admin.yourdomain.in
```

---

## 9. Process Management with PM2

### Step 9.1: Create PM2 Configuration

```bash
# Create PM2 ecosystem file
cd /var/www/mern-app
nano ecosystem.config.js
```

**PM2 Configuration File:**

```javascript
module.exports = {
  apps: [
    {
      name: "mern-backend",
      script: "./backend/server.js", // or app.js, index.js
      cwd: "/var/www/mern-app/backend",
      instances: 2, // or 'max' for all CPU cores
      exec_mode: "cluster",
      env: {
        NODE_ENV: "production",
        PORT: 3000,
      },
      error_file: "/var/www/mern-app/logs/backend-error.log",
      out_file: "/var/www/mern-app/logs/backend-out.log",
      log_file: "/var/www/mern-app/logs/backend-combined.log",
      time: true,
      max_memory_restart: "1G",
      node_args: "--max-old-space-size=1024",
      watch: false,
      ignore_watch: ["node_modules", "logs"],
      restart_delay: 5000,
      max_restarts: 5,
      min_uptime: "10s",
    },
  ],
};
```

### Step 9.2: Start Application with PM2

```bash
# Start the application
pm2 start ecosystem.config.js

# Check status
pm2 status

# View logs
pm2 logs mern-backend

# Save PM2 configuration
pm2 save

# Setup PM2 to start on system boot
pm2 startup
# Run the command provided by the above command
```

### Step 9.3: PM2 Management Commands

```bash
# Restart application
pm2 restart mern-backend

# Stop application
pm2 stop mern-backend

# Delete application from PM2
pm2 delete mern-backend

# Monitor in real-time
pm2 monit

# Reload with zero downtime
pm2 reload mern-backend

# View detailed info
pm2 show mern-backend

# Clear logs
pm2 flush
```

---

## 10. Security and Firewall

### Step 10.1: Configure AWS Security Groups

**Inbound Rules:**

```
Type: SSH, Port: 22, Source: Your IP
Type: HTTP, Port: 80, Source: 0.0.0.0/0
Type: HTTPS, Port: 443, Source: 0.0.0.0/0
Type: Custom TCP, Port: 3000, Source: 127.0.0.1/32 (localhost only)
```

### Step 10.2: Configure System Firewall

**For Amazon Linux:**

```bash
# Install and configure firewalld
sudo dnf install -y firewalld
sudo systemctl start firewalld
sudo systemctl enable firewalld

# Add firewall rules
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-service=ssh

# Reload firewall
sudo firewall-cmd --reload

# Check active rules
sudo firewall-cmd --list-all
```

**For Ubuntu:**

```bash
# UFW is usually pre-installed on Ubuntu
sudo ufw status

# If not installed, install it
sudo apt install -y ufw

# Configure UFW
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https

# Enable UFW
sudo ufw enable

# Check status
sudo ufw status verbose

# Alternative: Use firewalld on Ubuntu
# sudo apt install -y firewalld
# sudo systemctl start firewalld
# sudo systemctl enable firewalld
# (then use same commands as Amazon Linux)
```

### Step 10.3: Secure MongoDB Connection

**In your backend .env file:**

```env
# Use MongoDB Atlas with IP whitelist
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/database?retryWrites=true&w=majority

# Add your EC2 instance IP to MongoDB Atlas Network Access
```

### Step 10.4: Setup Basic Security

**For Amazon Linux:**

```bash
# Create basic auth for admin (optional)
sudo dnf install -y httpd-tools
sudo htpasswd -c /etc/nginx/.htpasswd admin

# Set permissions
sudo chmod 644 /etc/nginx/.htpasswd
```

**For Ubuntu:**

```bash
# Create basic auth for admin (optional)
sudo apt install -y apache2-utils
sudo htpasswd -c /etc/nginx/.htpasswd admin

# Set permissions
sudo chmod 644 /etc/nginx/.htpasswd
```

---

## 11. Monitoring and Maintenance

### Step 11.1: Setup Log Rotation

```bash
# Create logrotate configuration
sudo nano /etc/logrotate.d/mern-app
```

**Logrotate Configuration:**

```
/var/www/mern-app/logs/*.log {
    daily
    missingok
    rotate 52
    compress
    delaycompress
    notifempty
    copytruncate
    su ec2-user ec2-user  # For Amazon Linux
    # su ubuntu ubuntu    # For Ubuntu - uncomment this and comment above line
}
```

### Step 11.2: Create Deployment Script

```bash
# Create deployment script
nano /var/www/mern-app/scripts/deploy.sh
```

**Deployment Script:**

```bash
#!/bin/bash

# MERN App Deployment Script
set -e

APP_DIR="/var/www/mern-app"
BACKEND_DIR="$APP_DIR/backend"
FRONTEND_DIR="$APP_DIR/frontend"

echo "🚀 Starting deployment..."

# Pull latest changes
echo "📥 Pulling latest changes..."
cd $BACKEND_DIR && git pull origin main
cd $FRONTEND_DIR && git pull origin main

# Backend deployment
echo "🔧 Updating backend..."
cd $BACKEND_DIR
npm ci --production
pm2 restart mern-backend

# Frontend deployment
echo "⚛️  Building frontend..."
cd $FRONTEND_DIR
npm ci
npm run build

# Reload Nginx
echo "🔄 Reloading Nginx..."
sudo systemctl reload nginx

echo "✅ Deployment completed successfully!"

# Check application status
echo "📊 Application status:"
pm2 status
```

```bash
# Make script executable
chmod +x /var/www/mern-app/scripts/deploy.sh
```

### Step 11.3: Setup Monitoring

**For Amazon Linux:**

```bash
# Install htop for system monitoring
sudo dnf install -y htop

# Setup disk usage monitoring
echo "# Check disk usage daily" | sudo tee -a /etc/crontab
echo "0 2 * * * root df -h | mail -s 'Daily Disk Usage Report' your-email@example.com" | sudo tee -a /etc/crontab
```

**For Ubuntu:**

```bash
# Install htop for system monitoring
sudo apt install -y htop

# Setup disk usage monitoring
echo "# Check disk usage daily" | sudo tee -a /etc/crontab
echo "0 2 * * * root df -h | mail -s 'Daily Disk Usage Report' your-email@example.com" | sudo tee -a /etc/crontab
```

### Step 11.4: Backup Script

```bash
# Create backup script
nano /var/www/mern-app/scripts/backup.sh
```

**Backup Script:**

```bash
#!/bin/bash

# MERN App Backup Script
BACKUP_DIR="/var/www/backups"
APP_DIR="/var/www/mern-app"
DATE=$(date +%Y%m%d_%H%M%S)

# Create backup directory
mkdir -p $BACKUP_DIR

# Backup application files
echo "📦 Creating backup..."
tar -czf "$BACKUP_DIR/mern-app-$DATE.tar.gz" -C /var/www mern-app

# Keep only 7 days of backups
find $BACKUP_DIR -name "mern-app-*.tar.gz" -mtime +7 -delete

echo "✅ Backup completed: mern-app-$DATE.tar.gz"
```

```bash
# Make backup script executable
chmod +x /var/www/mern-app/scripts/backup.sh

# Setup daily backup cron job
crontab -e
# Add: 0 3 * * * /var/www/mern-app/scripts/backup.sh
```

---

## Common Commands Reference

### Git Operations

```bash
# Pull latest changes
git pull origin main

# Check status
git status

# View commit history
git log --oneline -10
```

### PM2 Operations

```bash
# Start app
pm2 start ecosystem.config.js

# Restart app
pm2 restart mern-backend

# View logs
pm2 logs mern-backend --lines 100

# Monitor
pm2 monit
```

### Nginx Operations

```bash
# Test configuration
sudo nginx -t

# Reload configuration
sudo systemctl reload nginx

# Restart Nginx
sudo systemctl restart nginx

# View error logs
sudo tail -f /var/log/nginx/error.log
```

### System Monitoring

```bash
# Check disk usage
df -h

# Check memory usage
free -h

# Check running processes
htop

# Check network connections
netstat -tulpn
```

### SSL Certificate Management

```bash
# Check certificate expiry
sudo certbot certificates

# Renew certificates
sudo certbot renew

# Test renewal
sudo certbot renew --dry-run
```

---

## Troubleshooting

### Common Issues and Solutions

1. **502 Bad Gateway Error**

   ```bash
   # Check if backend is running
   pm2 status

   # Check Nginx error logs
   sudo tail -f /var/log/nginx/error.log

   # Restart services
   pm2 restart mern-backend
   sudo systemctl restart nginx
   ```

2. **DNS Not Resolving**

   ```bash
   # Check DNS configuration
   nslookup yourdomain.in

   # Wait for DNS propagation (up to 48 hours)
   # Check with different DNS servers
   nslookup yourdomain.in 8.8.8.8
   ```

3. **SSL Certificate Issues**

   ```bash
   # Check certificate status
   sudo certbot certificates

   # Renew certificates
   sudo certbot renew --force-renewal
   ```

4. **Application Won't Start**

   ```bash
   # Check application logs
   pm2 logs mern-backend

   # Check for syntax errors
   cd /var/www/mern-app/backend
   node server.js

   # Check environment variables
   cat .env
   ```

5. **Database Connection Issues**
   ```bash
   # Test MongoDB connection
   cd /var/www/mern-app/backend
   node -e "
   require('dotenv').config();
   const mongoose = require('mongoose');
   mongoose.connect(process.env.MONGODB_URI)
     .then(() => console.log('✅ MongoDB connected'))
     .catch(err => console.error('❌ MongoDB error:', err));
   "
   ```

---

## Security Checklist

- [ ] SSH key-based authentication only
- [ ] Firewall configured properly
- [ ] SSL certificates installed and auto-renewing
- [ ] Environment variables secured
- [ ] MongoDB connection secured
- [ ] Regular backups scheduled
- [ ] Log rotation configured
- [ ] System updates automated
- [ ] Rate limiting configured
- [ ] Security headers added

---

## Performance Optimization

### Backend Optimization

```bash
# Enable Node.js production optimizations
export NODE_ENV=production

# Use PM2 cluster mode
pm2 start ecosystem.config.js --env production
```

### Frontend Optimization

```bash
# Optimize build
npm run build

# Enable gzip compression in Nginx (already included in config)
# Use CDN for static assets
# Implement caching strategies
```

### Database Optimization

```javascript
// In your backend code, add database indexing
// Create compound indexes for frequently queried fields
// Use connection pooling
// Implement query optimization
```

This comprehensive guide covers the complete process of deploying a MERN stack application on AWS EC2 with custom domain configuration. Follow each step carefully and customize the configurations according to your specific requirements.
