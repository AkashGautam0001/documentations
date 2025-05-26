# Complete AWS EC2 MERN Deployment Guide with GoDaddy Domain

## Table of Contents

1. [AWS EC2 Instance Setup](#aws-ec2-instance-setup)
2. [Server Configuration](#server-configuration)
3. [Project Deployment](#project-deployment)
4. [Domain Configuration](#domain-configuration)
5. [SSL Certificate Setup](#ssl-certificate-setup)
6. [Environment Management](#environment-management)
7. [Production Best Practices](#production-best-practices)
8. [Maintenance and Updates](#maintenance-and-updates)

## Prerequisites

- AWS Account (Free Tier eligible)
- GoDaddy Domain
- GitHub Repository with MERN project
- MongoDB connection string

---

## AWS EC2 Instance Setup

### Step 1: Launch EC2 Instance

#### 1.1 Access AWS Console

```bash
# Navigate to AWS Console
# Go to EC2 Dashboard
# Click "Launch Instance"
```

#### 1.2 Instance Configuration

- **Name**: `mern-production-server`
- **AMI**: Ubuntu Server 22.04 LTS (Free tier eligible)
- **Instance Type**: `t2.micro` (Free tier - 1 vCPU, 1 GB RAM)
- **Key Pair**: Create new key pair
  - Name: `mern-server-key`
  - Type: RSA
  - Format: .pem
  - **Download and save securely**

#### 1.3 Network Settings

- **VPC**: Default VPC
- **Subnet**: Default subnet
- **Auto-assign Public IP**: Enable
- **Security Group**: Create new
  - Name: `mern-security-group`
  - Description: `Security group for MERN application`

#### 1.4 Security Group Rules

```
Inbound Rules:
- SSH (22) - Your IP/Anywhere (0.0.0.0/0)
- HTTP (80) - Anywhere (0.0.0.0/0)
- HTTPS (443) - Anywhere (0.0.0.0/0)
- Custom TCP (3000) - Anywhere (0.0.0.0/0) [For development]
- Custom TCP (5000) - Anywhere (0.0.0.0/0) [For API]

Outbound Rules:
- All traffic (0.0.0.0/0)
```

#### 1.5 Storage

- **Root Volume**: 8 GB gp2 (Free tier limit)

### Step 2: Connect to Instance

#### 2.1 SSH Connection

```bash
# Change key permissions
chmod 400 mern-server-key.pem

# Connect to instance
ssh -i "mern-server-key.pem" ubuntu@your-ec2-public-ip

# Example:
ssh -i "mern-server-key.pem" ubuntu@54.123.456.789
```

---

## Server Configuration

### Step 3: Initial Server Setup

#### 3.1 Update System

```bash
# Update package list
sudo apt update

# Upgrade packages
sudo apt upgrade -y

# Install essential packages
sudo apt install -y curl wget git unzip software-properties-common
```

#### 3.2 Install Node.js

```bash
# Install Node.js 18.x LTS
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Verify installation
node --version
npm --version
```

#### 3.3 Install PM2 (Process Manager)

```bash
# Install PM2 globally
sudo npm install -g pm2

# Verify installation
pm2 --version
```

#### 3.4 Install Nginx

```bash
# Install Nginx
sudo apt install -y nginx

# Start and enable Nginx
sudo systemctl start nginx
sudo systemctl enable nginx

# Check status
sudo systemctl status nginx
```

#### 3.5 Configure Firewall

```bash
# Configure UFW firewall
sudo ufw enable
sudo ufw allow ssh
sudo ufw allow 'Nginx Full'
sudo ufw allow 3000
sudo ufw allow 5000

# Check status
sudo ufw status
```

---

## Project Deployment

### Step 4: Project Structure Setup

#### 4.1 Create Project Directory

```bash
# Navigate to web directory
cd /var/www

# Create project directory
sudo mkdir -p mern-app
sudo chown -R ubuntu:ubuntu mern-app
cd mern-app
```

#### 4.2 Clone Repository

```bash
# Clone your project
git clone https://github.com/yourusername/your-mern-repo.git .

# If private repository, configure SSH key or use token
# git clone https://username:token@github.com/yourusername/your-mern-repo.git .
```

### Step 5: Backend Setup

#### 5.1 Backend Directory Structure

```bash
# Navigate to backend directory
cd /var/www/mern-app/backend  # or wherever your backend is located

# Install dependencies
npm install

# Install production dependencies
npm install --production
```

#### 5.2 Environment Configuration

```bash
# Create .env file
nano .env
```

**Sample .env file:**

```env
NODE_ENV=production
PORT=5000
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/database
JWT_SECRET=your-super-secret-jwt-key-here-make-it-long-and-complex
CORS_ORIGIN=https://yourdomain.com,https://admin.yourdomain.com

# Email configuration (if using)
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USER=your-email@gmail.com
EMAIL_PASS=your-app-password

# Other API keys
STRIPE_SECRET_KEY=sk_live_...
CLOUDINARY_CLOUD_NAME=your-cloud-name
CLOUDINARY_API_KEY=your-api-key
CLOUDINARY_API_SECRET=your-api-secret
```

#### 5.3 Start Backend with PM2

```bash
# Create PM2 ecosystem file
nano ecosystem.config.js
```

**ecosystem.config.js:**

```javascript
module.exports = {
  apps: [
    {
      name: "mern-backend",
      script: "server.js", // or app.js, index.js - your main file
      cwd: "/var/www/mern-app/backend",
      instances: 1,
      autorestart: true,
      watch: false,
      max_memory_restart: "1G",
      env: {
        NODE_ENV: "production",
        PORT: 5000,
      },
      error_file: "/var/log/pm2/backend-error.log",
      out_file: "/var/log/pm2/backend-out.log",
      log_file: "/var/log/pm2/backend.log",
    },
  ],
};
```

```bash
# Create log directory
sudo mkdir -p /var/log/pm2
sudo chown -R ubuntu:ubuntu /var/log/pm2

# Start application
pm2 start ecosystem.config.js

# Save PM2 configuration
pm2 save

# Setup PM2 startup
pm2 startup
# Follow the instructions provided by the command
```

### Step 6: Frontend Setup

#### 6.1 Frontend Configuration

```bash
# Navigate to frontend directory
cd /var/www/mern-app/frontend  # or client directory

# Install dependencies
npm install
```

#### 6.2 Update API URLs

```bash
# Edit your API configuration file
nano src/config/api.js  # or wherever you store API URLs
```

**Update API URLs:**

```javascript
const config = {
  development: {
    API_URL: "http://localhost:5000/api",
  },
  production: {
    API_URL: "https://backend.yourdomain.com/api",
  },
};

export default config[process.env.NODE_ENV || "production"];
```

#### 6.3 Build Frontend

```bash
# Build for production
npm run build

# Move build files to web directory
sudo mkdir -p /var/www/html/main
sudo mkdir -p /var/www/html/admin
sudo cp -r build/* /var/www/html/main/

# If you have admin panel
# sudo cp -r admin-build/* /var/www/html/admin/
```

---

## Domain Configuration

### Step 7: GoDaddy DNS Setup

#### 7.1 Access GoDaddy DNS Management

1. Login to GoDaddy account
2. Go to "My Products" > "DNS"
3. Select your domain

#### 7.2 Configure DNS Records

**Add the following DNS records:**

| Type | Name    | Value              | TTL |
| ---- | ------- | ------------------ | --- |
| A    | @       | your-ec2-public-ip | 600 |
| A    | www     | your-ec2-public-ip | 600 |
| A    | admin   | your-ec2-public-ip | 600 |
| A    | backend | your-ec2-public-ip | 600 |
| A    | api     | your-ec2-public-ip | 600 |

**Example:**

```
A     @           54.123.456.789    600
A     www         54.123.456.789    600
A     admin       54.123.456.789    600
A     backend     54.123.456.789    600
A     api         54.123.456.789    600
```

### Step 8: Nginx Configuration

#### 8.1 Create Nginx Configuration

```bash
# Remove default configuration
sudo rm /etc/nginx/sites-enabled/default

# Create main site configuration
sudo nano /etc/nginx/sites-available/yourdomain.com
```

**Main Domain Configuration (/etc/nginx/sites-available/yourdomain.com):**

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    root /var/www/html/main;
    index index.html index.htm;

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml text/javascript application/javascript application/xml+rss application/json;

    # Main site
    location / {
        try_files $uri $uri/ /index.html;

        # Cache static assets
        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
    }

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;
}
```

#### 8.2 Admin Subdomain Configuration

```bash
# Create admin subdomain configuration
sudo nano /etc/nginx/sites-available/admin.yourdomain.com
```

**Admin Configuration:**

```nginx
server {
    listen 80;
    server_name admin.yourdomain.com;

    root /var/www/html/admin;
    index index.html index.htm;

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml text/javascript application/javascript application/xml+rss application/json;

    location / {
        try_files $uri $uri/ /index.html;

        # Cache static assets
        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
    }

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
}
```

#### 8.3 Backend Subdomain Configuration

```bash
# Create backend subdomain configuration
sudo nano /etc/nginx/sites-available/backend.yourdomain.com
```

**Backend Configuration:**

```nginx
server {
    listen 80;
    server_name backend.yourdomain.com api.yourdomain.com;

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;

    location / {
        limit_req zone=api burst=20 nodelay;

        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;

        # Timeout settings
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
}
```

#### 8.4 Enable Sites

```bash
# Enable all sites
sudo ln -s /etc/nginx/sites-available/yourdomain.com /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/admin.yourdomain.com /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/backend.yourdomain.com /etc/nginx/sites-enabled/

# Test Nginx configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx
```

---

## SSL Certificate Setup

### Step 9: Install and Configure SSL

#### 9.1 Install Certbot

```bash
# Install Certbot
sudo apt install -y certbot python3-certbot-nginx

# Or install via snap (alternative)
# sudo snap install --classic certbot
# sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

#### 9.2 Obtain SSL Certificates

```bash
# Get certificate for main domain
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Get certificate for admin subdomain
sudo certbot --nginx -d admin.yourdomain.com

# Get certificate for backend subdomain
sudo certbot --nginx -d backend.yourdomain.com -d api.yourdomain.com

# Or get all at once
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com -d admin.yourdomain.com -d backend.yourdomain.com -d api.yourdomain.com
```

#### 9.3 Auto-renewal Setup

```bash
# Test auto-renewal
sudo certbot renew --dry-run

# Check crontab for auto-renewal (should be automatically added)
sudo crontab -l

# If not present, add manually
sudo crontab -e
# Add this line:
# 0 12 * * * /usr/bin/certbot renew --quiet
```

---

## Environment Management

### Step 10: Managing Environment Variables

#### 10.1 Secure Environment File

```bash
# Set proper permissions for .env file
sudo chmod 600 /var/www/mern-app/backend/.env
sudo chown ubuntu:ubuntu /var/www/mern-app/backend/.env
```

#### 10.2 Environment Update Script

```bash
# Create update script
nano ~/update-env.sh
```

**Update Environment Script:**

```bash
#!/bin/bash

echo "Updating environment variables..."

# Backup current .env
cp /var/www/mern-app/backend/.env /var/www/mern-app/backend/.env.backup.$(date +%Y%m%d_%H%M%S)

# Edit .env file
nano /var/www/mern-app/backend/.env

# Restart application
pm2 restart mern-backend

echo "Environment updated and application restarted!"
```

```bash
# Make script executable
chmod +x ~/update-env.sh
```

#### 10.3 Quick Environment Updates

```bash
# Update specific environment variable
echo "NEW_VARIABLE=new_value" >> /var/www/mern-app/backend/.env

# Or edit directly
nano /var/www/mern-app/backend/.env

# Restart application after changes
pm2 restart mern-backend
```

---

## Production Best Practices

### Step 11: Security Hardening

#### 11.1 SSH Security

```bash
# Edit SSH configuration
sudo nano /etc/ssh/sshd_config
```

**Update SSH settings:**

```
Port 2200  # Change default port
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AllowUsers ubuntu
```

```bash
# Restart SSH service
sudo systemctl restart ssh

# Update security group to allow new SSH port
# AWS Console > EC2 > Security Groups > Add Rule: Custom TCP 2200
```

#### 11.2 Fail2Ban Installation

```bash
# Install Fail2Ban
sudo apt install -y fail2ban

# Create local configuration
sudo nano /etc/fail2ban/jail.local
```

**Fail2Ban Configuration:**

```ini
[DEFAULT]
bantime = 10m
findtime = 10m
maxretry = 5

[sshd]
enabled = true
port = 2200
logpath = /var/log/auth.log

[nginx-http-auth]
enabled = true
filter = nginx-http-auth
logpath = /var/log/nginx/error.log

[nginx-limit-req]
enabled = true
filter = nginx-limit-req
logpath = /var/log/nginx/error.log
```

```bash
# Start and enable Fail2Ban
sudo systemctl start fail2ban
sudo systemctl enable fail2ban
```

#### 11.3 MongoDB Security

```bash
# If using local MongoDB (not recommended for production)
# Secure MongoDB configuration
sudo nano /etc/mongod.conf
```

**MongoDB Security (if local):**

```yaml
security:
  authorization: enabled
net:
  bindIpAll: false
  bindIp: 127.0.0.1
```

### Step 12: Monitoring Setup

#### 12.1 Log Management

```bash
# Create log rotation for PM2
sudo nano /etc/logrotate.d/pm2
```

**Log Rotation Configuration:**

```
/var/log/pm2/*.log {
    daily
    missingok
    rotate 30
    compress
    notifempty
    create 0644 ubuntu ubuntu
    postrotate
        sudo -u ubuntu pm2 reloadLogs
    endscript
}
```

#### 12.2 System Monitoring

```bash
# Install htop for monitoring
sudo apt install -y htop

# Check system resources
htop

# Check disk usage
df -h

# Check memory usage
free -h

# Check PM2 processes
pm2 status
pm2 logs
pm2 monit
```

---

## Maintenance and Updates

### Step 13: Code Updates and Deployment

#### 13.1 Update Script

```bash
# Create deployment script
nano ~/deploy.sh
```

**Deployment Script:**

```bash
#!/bin/bash

set -e  # Exit on any error

echo "Starting deployment..."

# Navigate to project directory
cd /var/www/mern-app

# Backup current version
BACKUP_DIR="/var/backups/mern-app/$(date +%Y%m%d_%H%M%S)"
sudo mkdir -p /var/backups/mern-app
sudo cp -r /var/www/mern-app $BACKUP_DIR
echo "Backup created at: $BACKUP_DIR"

# Pull latest changes
git stash  # Stash any local changes
git pull origin main  # or your main branch

# Backend updates
echo "Updating backend..."
cd backend
npm install --production

# Frontend updates
echo "Updating frontend..."
cd ../frontend
npm install
npm run build

# Update frontend files
sudo rm -rf /var/www/html/main/*
sudo cp -r build/* /var/www/html/main/

# If admin panel exists
# sudo rm -rf /var/www/html/admin/*
# sudo cp -r admin-build/* /var/www/html/admin/

# Restart backend
echo "Restarting backend..."
pm2 restart mern-backend

# Clear Nginx cache (if using)
sudo systemctl reload nginx

echo "Deployment completed successfully!"
echo "Backup available at: $BACKUP_DIR"
```

```bash
# Make script executable
chmod +x ~/deploy.sh
```

#### 13.2 Quick Updates

```bash
# Update backend only
cd /var/www/mern-app/backend
git pull origin main
npm install --production
pm2 restart mern-backend

# Update frontend only
cd /var/www/mern-app/frontend
git pull origin main
npm install
npm run build
sudo cp -r build/* /var/www/html/main/
```

#### 13.3 Environment Variable Updates

```bash
# Method 1: Direct edit
nano /var/www/mern-app/backend/.env
pm2 restart mern-backend

# Method 2: Using update script
~/update-env.sh

# Method 3: Replace entire file
sudo cp /path/to/new/.env /var/www/mern-app/backend/.env
pm2 restart mern-backend
```

### Step 14: Backup Strategy

#### 14.1 Automated Backup Script

```bash
# Create backup script
nano ~/backup.sh
```

**Backup Script:**

```bash
#!/bin/bash

# Configuration
BACKUP_DIR="/var/backups/mern-app"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=7

# Create backup directory
mkdir -p $BACKUP_DIR

# Backup application files
tar -czf "$BACKUP_DIR/app-$DATE.tar.gz" -C /var/www mern-app

# Backup Nginx configuration
tar -czf "$BACKUP_DIR/nginx-$DATE.tar.gz" -C /etc/nginx sites-available sites-enabled

# Backup PM2 configuration
tar -czf "$BACKUP_DIR/pm2-$DATE.tar.gz" -C ~/.pm2 dump.pm2

# Clean old backups
find $BACKUP_DIR -name "*.tar.gz" -mtime +$RETENTION_DAYS -delete

echo "Backup completed: $DATE"
```

```bash
# Make executable
chmod +x ~/backup.sh

# Add to crontab for daily backup
crontab -e
# Add: 0 2 * * * /home/ubuntu/backup.sh
```

### Step 15: Troubleshooting

#### 15.1 Common Issues and Solutions

**Backend not starting:**

```bash
# Check PM2 logs
pm2 logs mern-backend

# Check if port is in use
sudo netstat -tulpn | grep :5000

# Restart PM2
pm2 restart mern-backend

# If still issues, start manually for debugging
cd /var/www/mern-app/backend
node server.js
```

**Frontend not loading:**

```bash
# Check Nginx logs
sudo tail -f /var/log/nginx/error.log
sudo tail -f /var/log/nginx/access.log

# Test Nginx configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx

# Check file permissions
ls -la /var/www/html/main/
```

**SSL certificate issues:**

```bash
# Check certificate status
sudo certbot certificates

# Renew certificates
sudo certbot renew

# Test SSL configuration
openssl s_client -connect yourdomain.com:443
```

**Domain not resolving:**

```bash
# Check DNS propagation
nslookup yourdomain.com
dig yourdomain.com

# Check from different locations
# Use online tools like whatsmydns.net
```

#### 15.2 Monitoring Commands

```bash
# System monitoring
htop                    # System resources
pm2 status             # PM2 processes
pm2 logs               # Application logs
sudo systemctl status nginx  # Nginx status
df -h                  # Disk usage
free -h                # Memory usage

# Network monitoring
sudo netstat -tulpn    # Open ports
sudo ss -tulpn         # Modern alternative

# Log monitoring
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log
tail -f /var/log/auth.log
```

### Step 16: Scaling Considerations

#### 16.1 Free Tier Limitations

- **t2.micro**: 1 vCPU, 1 GB RAM
- **Storage**: 8 GB (can be increased)
- **Bandwidth**: 15 GB outbound per month
- **Data Transfer**: 1 GB outbound to internet per month

#### 16.2 Performance Optimization

```bash
# Enable Nginx caching
sudo nano /etc/nginx/nginx.conf
```

**Add caching configuration:**

```nginx
http {
    # Enable caching
    proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m max_size=10g inactive=60m use_temp_path=off;

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
}
```

#### 16.3 Database Optimization

```bash
# For MongoDB Atlas (recommended)
# Use connection pooling in your application
# Enable compression in connection string
# Example: mongodb+srv://user:pass@cluster.mongodb.net/db?retryWrites=true&w=majority&compressors=zlib
```

## Final Checklist

- [ ] EC2 instance created and configured
- [ ] Node.js and PM2 installed
- [ ] Nginx installed and configured
- [ ] Project cloned and deployed
- [ ] Environment variables configured
- [ ] DNS records configured in GoDaddy
- [ ] SSL certificates installed
- [ ] Security measures implemented
- [ ] Monitoring set up
- [ ] Backup strategy implemented
- [ ] Deployment scripts created

## Useful Commands Summary

```bash
# Application Management
pm2 start ecosystem.config.js  # Start application
pm2 restart mern-backend       # Restart application
pm2 stop mern-backend         # Stop application
pm2 logs mern-backend         # View logs
pm2 status                    # Check status

# Nginx Management
sudo nginx -t                 # Test configuration
sudo systemctl reload nginx   # Reload configuration
sudo systemctl restart nginx  # Restart Nginx

# SSL Management
sudo certbot renew           # Renew certificates
sudo certbot certificates    # List certificates

# System Updates
sudo apt update && sudo apt upgrade -y  # Update system
~/deploy.sh                 # Deploy application
~/backup.sh                 # Create backup
```

This documentation provides a complete production-ready setup for deploying a MERN application on AWS EC2 with proper domain configuration, SSL certificates, and security measures. Follow each step carefully and customize according to your specific project requirements.
