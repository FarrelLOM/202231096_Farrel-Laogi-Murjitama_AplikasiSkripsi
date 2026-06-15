# 🚀 Panduan Deployment - Proctoring Application

Dokumen ini berisi panduan lengkap untuk men-deploy aplikasi ke berbagai platform production.

---

## 📋 Deployment Options Overview

| Platform | Difficulty | Cost | Best For | Model Support |
|----------|-----------|------|----------|----------------|
| **Vercel** | ⭐ Easy | Free-$20/mo | Frontend/API ringan | ❌ No (500MB limit) |
| **Heroku** | ⭐⭐ Medium | $50+/mo | Small-medium apps | ⚠️ Limited |
| **AWS** | ⭐⭐⭐ Hard | $50-500/mo | Scalable, production | ✅ Full support |
| **Digital Ocean** | ⭐⭐ Medium | $5-50/mo | Affordable production | ✅ Full support |
| **VPS (Contabo/Linode)** | ⭐⭐⭐ Hard | $3-15/mo | Budget production | ✅ Full support |
| **Docker + K8s** | ⭐⭐⭐⭐ Very Hard | Variable | Enterprise | ✅ Full support |

**⚠️ CATATAN PENTING**: Vercel TIDAK cocok untuk deployment penuh. Model ML terlalu besar (460MB). Gunakan untuk frontend saja atau API gateway.

---

## 🔧 Pre-Deployment Checklist

```bash
# 1. Test aplikasi lokal
python test_system.py

# 2. Verifikasi database
python verify_structure.py

# 3. Test semua endpoints
python test_violation_api_fixed.py

# 4. Check file sizes
du -sh .
du -sh gaze_tracking/trained_models/

# 5. Cleanup cache
find . -type d -name __pycache__ -exec rm -rf {} +
find . -type f -name "*.pyc" -delete

# 6. Update requirements
pip freeze | grep -v "^-e" > requirements.txt

# 7. Security check
# - Verify SECRET_KEY is strong
# - Check no passwords in code
# - Verify .env in .gitignore
```

---

## ⭐ RECOMMENDED: Digital Ocean Deployment

Digital Ocean adalah pilihan terbaik untuk balance antara cost, ease, dan performance.

### Step 1: Setup Digital Ocean Droplet

#### A. Create Droplet
1. Login ke [DigitalOcean Dashboard](https://cloud.digitalocean.com)
2. Click "Create" → "Droplets"
3. Choose Specifications:
   - **Image**: Ubuntu 22.04 LTS
   - **Plan**: $4-6/mo (1GB RAM, 1 CPU, 25GB SSD)
   - **Region**: Pilih terdekat dengan user (sgp1 untuk Indonesia)
   - **Authentication**: SSH Key (recommended) atau Password
4. Click "Create Droplet"

#### B. Initial Server Setup

```bash
# SSH ke server
ssh root@YOUR_DROPLET_IP

# Update system
apt update && apt upgrade -y

# Create new user (recommended, don't use root)
adduser proctoring
usermod -aG sudo proctoring
su - proctoring

# Generate SSH key (optional)
ssh-keygen -t rsa -b 4096
```

### Step 2: Install Dependencies

```bash
# Install system packages
sudo apt install -y \
  python3.10 \
  python3-pip \
  python3-venv \
  mysql-server \
  nginx \
  git \
  curl \
  wget

# Start services
sudo systemctl start mysql
sudo systemctl enable mysql
sudo systemctl start nginx
sudo systemctl enable nginx

# Verify installations
python3 --version
mysql --version
nginx -v
```

### Step 3: Clone & Setup Project

```bash
# Create app directory
mkdir -p /home/proctoring/apps
cd /home/proctoring/apps

# Clone repository
git clone https://github.com/yourusername/proctoring-app.git
cd proctoring-app

# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Upgrade pip
pip install --upgrade pip setuptools wheel

# Install dependencies
pip install -r requirements.txt

# Takes ~10-15 minutes depending on internet speed
```

### Step 4: Configure Environment

```bash
# Copy .env.example to .env
cp .env.example .env

# Edit .env
nano .env
```

```ini
# Server
SERVER_HOST=0.0.0.0
PORT=5000
SECRET_KEY=generate-very-secure-key-here

# Database
DB_HOST=localhost
DB_PORT=3306
DB_NAME=proctoring_app
DB_USER=proctoring
DB_PASSWORD=secure_db_password

# Admin
ADMIN_USERNAME=admin
ADMIN_PASSWORD=strong_password_here

# Cloudinary
CLOUDINARY_CLOUD_NAME=your_cloud
CLOUDINARY_API_KEY=your_key
CLOUDINARY_API_SECRET=your_secret
```

### Step 5: Setup MySQL Database

```bash
# Login ke MySQL
mysql -u root

# Di MySQL prompt:
CREATE DATABASE proctoring_app;
CREATE USER 'proctoring'@'localhost' IDENTIFIED BY 'secure_db_password';
GRANT ALL PRIVILEGES ON proctoring_app.* TO 'proctoring'@'localhost';
FLUSH PRIVILEGES;
EXIT;

# Run migration
mysql -u proctoring -p proctoring_app < DATABASE_MIGRATION.sql
# Enter password: secure_db_password
```

### Step 6: Setup Gunicorn (WSGI Server)

```bash
# Install Gunicorn
pip install gunicorn

# Test run
gunicorn -w 4 -b 127.0.0.1:5000 app:app

# Verify berfungsi, then Ctrl+C to stop
```

### Step 7: Create Systemd Service

```bash
# Create service file
sudo nano /etc/systemd/system/proctoring.service
```

```ini
[Unit]
Description=Proctoring Application
After=network.target mysql.service

[Service]
User=proctoring
WorkingDirectory=/home/proctoring/apps/proctoring-app
ExecStart=/home/proctoring/apps/proctoring-app/venv/bin/gunicorn \
    -w 4 \
    -b 127.0.0.1:5000 \
    --timeout 120 \
    --access-logfile /home/proctoring/apps/proctoring-app/logs/access.log \
    --error-logfile /home/proctoring/apps/proctoring-app/logs/error.log \
    app:app

Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

```bash
# Enable dan start service
sudo systemctl daemon-reload
sudo systemctl enable proctoring
sudo systemctl start proctoring

# Verify status
sudo systemctl status proctoring

# View logs
sudo journalctl -u proctoring -f
```

### Step 8: Setup Nginx (Reverse Proxy)

```bash
# Create Nginx config
sudo nano /etc/nginx/sites-available/proctoring
```

```nginx
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;

    # Redirect HTTP to HTTPS (after SSL setup)
    # return 301 https://$server_name$request_uri;

    client_max_body_size 50M;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_buffering off;
        proxy_request_buffering off;
    }

    location /static/ {
        alias /home/proctoring/apps/proctoring-app/static/;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # Log files
    access_log /var/log/nginx/proctoring_access.log;
    error_log /var/log/nginx/proctoring_error.log;
}
```

```bash
# Enable site
sudo ln -s /etc/nginx/sites-available/proctoring /etc/nginx/sites-enabled/

# Disable default site
sudo rm /etc/nginx/sites-enabled/default

# Test Nginx config
sudo nginx -t

# Restart Nginx
sudo systemctl restart nginx
```

### Step 9: Setup SSL Certificate (HTTPS)

```bash
# Install Certbot
sudo apt install -y certbot python3-certbot-nginx

# Get SSL certificate
sudo certbot --nginx -d your-domain.com -d www.your-domain.com

# Auto-renewal (runs automatically every 12 hours)
sudo systemctl enable certbot.timer
sudo systemctl start certbot.timer

# Verify renewal
sudo certbot renew --dry-run
```

### Step 10: Setup Firewall

```bash
# Enable UFW firewall
sudo ufw enable

# Allow SSH (CRITICAL - jangan lupa!)
sudo ufw allow 22/tcp

# Allow HTTP/HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Deny others
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Check status
sudo ufw status
```

### Step 11: Backup & Monitoring

#### Setup Database Backup

```bash
# Create backup script
mkdir -p /home/proctoring/backups
nano /home/proctoring/backups/backup.sh
```

```bash
#!/bin/bash
BACKUP_DIR="/home/proctoring/backups"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/proctoring_app_$DATE.sql"

# Backup database
mysqldump -u proctoring -p'secure_db_password' proctoring_app > "$BACKUP_FILE"

# Compress
gzip "$BACKUP_FILE"

# Keep only last 7 days
find "$BACKUP_DIR" -name "*.sql.gz" -mtime +7 -delete

echo "Backup completed: $BACKUP_FILE.gz"
```

```bash
# Make executable
chmod +x /home/proctoring/backups/backup.sh

# Add to crontab (daily 2 AM)
crontab -e
# Add: 0 2 * * * /home/proctoring/backups/backup.sh
```

#### Setup Log Rotation

```bash
# Create logs directory
mkdir -p /home/proctoring/apps/proctoring-app/logs

# Setup logrotate
sudo nano /etc/logrotate.d/proctoring
```

```
/home/proctoring/apps/proctoring-app/logs/*.log {
    daily
    rotate 14
    compress
    delaycompress
    notifempty
    create 0644 proctoring proctoring
    sharedscripts
    postrotate
        systemctl reload proctoring > /dev/null 2>&1 || true
    endscript
}
```

---

## AWS Deployment (EC2)

### Quick Setup

```bash
# 1. Create EC2 Instance
# - Choose Ubuntu 22.04 LTS
# - Instance type: t3.small (1 vCPU, 2GB RAM)
# - Security group: Allow 22, 80, 443

# 2. SSH ke instance
ssh -i your-key.pem ubuntu@YOUR_INSTANCE_IP

# 3. Follow same steps as Digital Ocean above
# (Server setup, Python, MySQL, Nginx, Gunicorn)

# 4. Differences:
# - RDS for database (optional)
# - S3 untuk backups
# - CloudFront untuk CDN
# - Route53 untuk DNS
```

---

## Docker Deployment

Untuk deployment yang lebih konsisten:

### Dockerfile

```dockerfile
FROM python:3.10-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    libglib2.0-0 \
    libsm6 \
    libxext6 \
    libxrender-dev \
    libopenblas-dev \
    mysql-client \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Expose port
EXPOSE 5000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
    CMD curl -f http://localhost:5000/health || exit 1

# Run Gunicorn
CMD ["gunicorn", "-w", "4", "-b", "0.0.0.0:5000", "--timeout", "120", "app:app"]
```

### docker-compose.yml

```yaml
version: '3.8'

services:
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: proctoring_app
    volumes:
      - mysql_data:/var/lib/mysql
      - ./DATABASE_MIGRATION.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "3306:3306"

  app:
    build: .
    ports:
      - "5000:5000"
    environment:
      DB_HOST: mysql
      DB_USER: root
      DB_PASSWORD: root_password
      DB_NAME: proctoring_app
    depends_on:
      - mysql
    volumes:
      - ./logs:/app/logs

volumes:
  mysql_data:
```

```bash
# Run dengan Docker Compose
docker-compose up -d

# View logs
docker-compose logs -f app

# Stop
docker-compose down
```

---

## Performance Tuning

### Optimize Gunicorn

```bash
# Calculate workers formula: 2 × CPU + 1
# Untuk t3.small (1 CPU): 2 × 1 + 1 = 3 workers

gunicorn -w 3 -b 0.0.0.0:5000 \
    --worker-class sync \
    --worker-connections 1000 \
    --max-requests 1000 \
    --max-requests-jitter 50 \
    --timeout 120 \
    app:app
```

### Optimize MySQL

```sql
# Increase max connections
SET GLOBAL max_connections = 1000;

# Optimize query cache (MySQL 5.7)
SET GLOBAL query_cache_size = 268435456;  # 256MB

# Save to my.cnf for persistence
sudo nano /etc/mysql/my.cnf
# Add:
# max_connections = 1000
# query_cache_size = 256M
```

### Optimize Nginx

```nginx
# Add ke /etc/nginx/nginx.conf
worker_connections 2048;
keepalive_timeout 65;

# Enable gzip compression
gzip on;
gzip_types text/plain text/css text/xml application/json;
gzip_min_length 1000;
```

---

## Monitoring & Alerts

### Setup Monitoring

```bash
# Install monitoring tools
sudo apt install -y htop iotop nethogs

# Monitor in real-time
htop

# Check disk usage
df -h

# Check service status
sudo systemctl status proctoring
sudo systemctl status nginx
sudo systemctl status mysql
```

### Syslog Configuration

```bash
# View application logs
sudo journalctl -u proctoring -n 100

# Real-time logs
sudo journalctl -u proctoring -f

# Filter by date
sudo journalctl -u proctoring --since "2024-01-20 10:00:00"
```

---

## Troubleshooting Deployment

### Service won't start

```bash
# Check service status
sudo systemctl status proctoring

# View logs
sudo journalctl -u proctoring -n 50 -e

# Check port availability
sudo lsof -i :5000

# Check file permissions
ls -la /home/proctoring/apps/proctoring-app/
```

### Database connection failed

```bash
# Test connection
mysql -h localhost -u proctoring -p

# Check MySQL service
sudo systemctl status mysql

# Check credentials in .env
cat /home/proctoring/apps/proctoring-app/.env | grep DB_
```

### SSL certificate issues

```bash
# Check certificate
sudo certbot certificates

# Manual renewal
sudo certbot renew --force-renewal

# Check expiry
curl -vI https://your-domain.com 2>&1 | grep expire
```

---

## Scaling Strategies

### Horizontal Scaling (Multiple Servers)

1. Load Balancer (HAProxy/Nginx)
2. Multiple Application Servers
3. Shared Database (Master-Slave)
4. Shared Storage (NFS/S3)

### Vertical Scaling (Bigger Server)

1. Increase CPU cores
2. Increase RAM
3. Upgrade storage
4. Upgrade network

---

## Security Best Practices

- [ ] Change default SSH port
- [ ] Disable root login
- [ ] Setup fail2ban for brute force protection
- [ ] Regular security updates: `sudo apt update && sudo apt upgrade`
- [ ] Configure firewall strictly
- [ ] Setup SSL/TLS
- [ ] Rotate SSH keys regularly
- [ ] Setup VPN for admin access
- [ ] Regular backups to separate location

---

## Cost Estimation (per month)

| Component | Provider | Cost |
|-----------|----------|------|
| Server (1GB RAM) | Digital Ocean | $4 |
| Database | Included | $0 |
| Backups | DIY | $0 |
| SSL | Let's Encrypt | $0 |
| **TOTAL** | | **$4** |

---

## Checklist Sebelum Go-Live

- [ ] All tests pass locally and on staging
- [ ] Database backup configured
- [ ] SSL certificate installed
- [ ] Firewall configured
- [ ] DNS records updated
- [ ] Admin can login
- [ ] Exam can start
- [ ] Violations recorded correctly
- [ ] Cloud storage working
- [ ] Email notifications working (if enabled)
- [ ] Monitoring setup
- [ ] Support plan ready

---

**Happy Deploying! 🚀**
