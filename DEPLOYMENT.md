# Deployment Guide

This guide covers deployment strategies and best practices for the AJ Care Motors platform.

## Table of Contents

- [Deployment Overview](#deployment-overview)
- [Pre-Deployment Checklist](#pre-deployment-checklist)
- [Production Environment Setup](#production-environment-setup)
- [Server Configuration](#server-configuration)
- [Database Setup](#database-setup)
- [SSL Configuration](#ssl-configuration)
- [Performance Optimization](#performance-optimization)
- [Monitoring and Logging](#monitoring-and-logging)
- [Backup Strategy](#backup-strategy)
- [CI/CD Pipeline](#cicd-pipeline)
- [Troubleshooting](#troubleshooting)

## Deployment Overview

### Deployment Options

1. **Shared Hosting** - Budget-friendly, limited control
2. **VPS/Cloud Servers** - Balanced cost and control
3. **Dedicated Servers** - Maximum performance and control
4. **Container Deployment** - Docker/Kubernetes
5. **Serverless** - AWS Lambda, Vercel, etc.

### Recommended Stack

- **Web Server**: Nginx or Apache
- **PHP**: Version 8.0+
- **Database**: MySQL 8.0+ or PostgreSQL 13+
- **Cache**: Redis
- **Queue**: Redis or Database
- **SSL**: Let's Encrypt or commercial certificate

## Pre-Deployment Checklist

### Code Preparation
- [ ] All tests passing
- [ ] Code reviewed and approved
- [ ] Dependencies updated and secure
- [ ] Environment-specific configurations set
- [ ] Database migrations tested
- [ ] Assets compiled for production

### Security Checklist
- [ ] Debug mode disabled (`APP_DEBUG=false`)
- [ ] Strong application key generated
- [ ] Database credentials secured
- [ ] HTTPS enabled
- [ ] CORS properly configured
- [ ] Input validation implemented
- [ ] Rate limiting enabled

### Performance Checklist
- [ ] Caching enabled
- [ ] Database queries optimized
- [ ] Images optimized
- [ ] CDN configured (if applicable)
- [ ] Compression enabled

## Production Environment Setup

### Server Requirements

#### Minimum Requirements
- **CPU**: 2 cores
- **RAM**: 4GB
- **Storage**: 50GB SSD
- **Bandwidth**: 100GB/month

#### Recommended Specifications
- **CPU**: 4+ cores
- **RAM**: 8GB+
- **Storage**: 100GB+ SSD
- **Bandwidth**: Unlimited

### Operating System Setup

#### Ubuntu 20.04/22.04 LTS
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install required packages
sudo apt install -y software-properties-common

# Add PHP repository
sudo add-apt-repository ppa:ondrej/php
sudo apt update

# Install PHP and extensions
sudo apt install -y php8.1 php8.1-fpm php8.1-mysql php8.1-xml php8.1-curl \
    php8.1-mbstring php8.1-zip php8.1-bcmath php8.1-intl php8.1-gd \
    php8.1-redis php8.1-opcache

# Install Composer
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt install -y nodejs

# Install MySQL
sudo apt install -y mysql-server

# Install Redis
sudo apt install -y redis-server

# Install Nginx
sudo apt install -y nginx

# Install Supervisor (for queue workers)
sudo apt install -y supervisor
```

## Server Configuration

### Nginx Configuration

Create `/etc/nginx/sites-available/ajcaremotors`:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name ajcaremotors.com www.ajcaremotors.com;
    root /var/www/ajcaremotors/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

Enable the site:
```bash
sudo ln -s /etc/nginx/sites-available/ajcaremotors /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### PHP Configuration

Edit `/etc/php/8.1/fpm/php.ini`:

```ini
; Basic settings
memory_limit = 256M
max_execution_time = 300
max_input_vars = 3000
upload_max_filesize = 64M
post_max_size = 64M

; OPcache settings
opcache.enable=1
opcache.enable_cli=1
opcache.memory_consumption=128
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=4000
opcache.revalidate_freq=2
opcache.fast_shutdown=1

; Security settings
expose_php = Off
display_errors = Off
log_errors = On
error_log = /var/log/php_errors.log
```

### PHP-FPM Configuration

Edit `/etc/php/8.1/fpm/pool.d/www.conf`:

```ini
[www]
user = www-data
group = www-data
listen = /var/run/php/php8.1-fpm.sock
listen.owner = www-data
listen.group = www-data
pm = dynamic
pm.max_children = 20
pm.start_servers = 2
pm.min_spare_servers = 1
pm.max_spare_servers = 3
pm.max_requests = 500
```

Restart PHP-FPM:
```bash
sudo systemctl restart php8.1-fpm
```

## Database Setup

### MySQL Configuration

```bash
# Secure MySQL installation
sudo mysql_secure_installation

# Create database and user
sudo mysql -u root -p
```

```sql
CREATE DATABASE ajcaremotors CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'ajcare_user'@'localhost' IDENTIFIED BY 'strong_password_here';
GRANT ALL PRIVILEGES ON ajcaremotors.* TO 'ajcare_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### MySQL Performance Tuning

Edit `/etc/mysql/mysql.conf.d/mysqld.cnf`:

```ini
[mysqld]
# Performance settings
innodb_buffer_pool_size = 2G
innodb_log_file_size = 256M
innodb_flush_log_at_trx_commit = 2
innodb_flush_method = O_DIRECT

# Connection settings
max_connections = 200
connect_timeout = 10
wait_timeout = 600

# Query cache (MySQL 5.7)
query_cache_type = 1
query_cache_size = 64M
```

## Application Deployment

### Deploy Application

```bash
# Create application directory
sudo mkdir -p /var/www/ajcaremotors
sudo chown $USER:www-data /var/www/ajcaremotors

# Clone repository
cd /var/www
git clone https://github.com/ruhulamin63/ajcaremotors.git
cd ajcaremotors

# Install dependencies
composer install --optimize-autoloader --no-dev
npm install
npm run production

# Set permissions
sudo chown -R www-data:www-data storage bootstrap/cache
sudo chmod -R 775 storage bootstrap/cache

# Create environment file
cp .env.example .env
```

### Environment Configuration

Edit `.env` for production:

```env
APP_NAME="AJ Care Motors"
APP_ENV=production
APP_KEY=base64:your_generated_key_here
APP_DEBUG=false
APP_URL=https://ajcaremotors.com

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=ajcaremotors
DB_USERNAME=ajcare_user
DB_PASSWORD=your_strong_password

CACHE_DRIVER=redis
SESSION_DRIVER=redis
QUEUE_CONNECTION=redis

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_MAILER=smtp
MAIL_HOST=your_smtp_host
MAIL_PORT=587
MAIL_USERNAME=your_email
MAIL_PASSWORD=your_password
MAIL_ENCRYPTION=tls
```

### Generate Application Key and Run Migrations

```bash
php artisan key:generate
php artisan migrate --force
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

## SSL Configuration

### Using Let's Encrypt (Certbot)

```bash
# Install Certbot
sudo apt install -y certbot python3-certbot-nginx

# Get SSL certificate
sudo certbot --nginx -d ajcaremotors.com -d www.ajcaremotors.com

# Test automatic renewal
sudo certbot renew --dry-run
```

### Nginx SSL Configuration

The SSL configuration will be automatically added by Certbot:

```nginx
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name ajcaremotors.com www.ajcaremotors.com;
    
    ssl_certificate /etc/letsencrypt/live/ajcaremotors.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/ajcaremotors.com/privkey.pem;
    
    # SSL settings
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    
    # Security headers
    add_header Strict-Transport-Security "max-age=63072000" always;
    add_header X-Frame-Options DENY always;
    add_header X-Content-Type-Options nosniff always;
    
    # Application configuration
    root /var/www/ajcaremotors/public;
    index index.php;
    
    # ... rest of configuration
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name ajcaremotors.com www.ajcaremotors.com;
    return 301 https://$server_name$request_uri;
}
```

## Queue Worker Setup

### Supervisor Configuration

Create `/etc/supervisor/conf.d/ajcaremotors-worker.conf`:

```ini
[program:ajcaremotors-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/ajcaremotors/artisan queue:work redis --sleep=3 --tries=3 --max-time=3600
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=www-data
numprocs=2
redirect_stderr=true
stdout_logfile=/var/www/ajcaremotors/storage/logs/worker.log
stopwaitsecs=3600
```

Start the worker:
```bash
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start ajcaremotors-worker:*
```

## Performance Optimization

### Enable Compression

Add to Nginx configuration:
```nginx
# Gzip compression
gzip on;
gzip_vary on;
gzip_min_length 1024;
gzip_proxied expired no-cache no-store private auth;
gzip_types
    text/plain
    text/css
    text/xml
    text/javascript
    application/javascript
    application/xml+rss
    application/json;
```

### Browser Caching

Add to Nginx configuration:
```nginx
# Browser caching
location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}
```

### Laravel Optimizations

```bash
# Cache configurations
php artisan config:cache
php artisan route:cache
php artisan view:cache

# Optimize autoloader
composer dump-autoload --optimize

# Enable OPcache (already configured in php.ini)
```

## Monitoring and Logging

### Log Configuration

Configure log rotation in `/etc/logrotate.d/ajcaremotors`:

```
/var/www/ajcaremotors/storage/logs/*.log {
    daily
    missingok
    rotate 52
    compress
    delaycompress
    notifempty
    create 644 www-data www-data
    postrotate
        sudo systemctl reload php8.1-fpm
    endscript
}
```

### System Monitoring

Install monitoring tools:
```bash
# Install htop for system monitoring
sudo apt install -y htop

# Install fail2ban for security
sudo apt install -y fail2ban

# Configure fail2ban for Nginx
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

### Application Monitoring

Consider integrating:
- **Laravel Telescope** (development)
- **Sentry** (error tracking)
- **New Relic** (performance monitoring)
- **Datadog** (infrastructure monitoring)

## Backup Strategy

### Database Backup Script

Create `/var/www/ajcaremotors/backup-db.sh`:

```bash
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/var/backups/ajcaremotors"
DB_NAME="ajcaremotors"
DB_USER="ajcare_user"
DB_PASS="your_password"

mkdir -p $BACKUP_DIR

# Create database backup
mysqldump -u $DB_USER -p$DB_PASS $DB_NAME | gzip > $BACKUP_DIR/db_backup_$DATE.sql.gz

# Remove backups older than 30 days
find $BACKUP_DIR -name "db_backup_*.sql.gz" -mtime +30 -delete

echo "Database backup completed: db_backup_$DATE.sql.gz"
```

### Automated Backups

Add to crontab:
```bash
# Edit crontab
sudo crontab -e

# Add backup jobs
0 2 * * * /var/www/ajcaremotors/backup-db.sh
0 3 * * 0 tar -czf /var/backups/ajcaremotors/files_$(date +\%Y\%m\%d).tar.gz /var/www/ajcaremotors
```

## CI/CD Pipeline

### GitHub Actions Example

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to Production

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Setup PHP
      uses: shivammathur/setup-php@v2
      with:
        php-version: '8.1'
        
    - name: Install dependencies
      run: composer install --optimize-autoloader --no-dev
      
    - name: Run tests
      run: php artisan test
      
    - name: Deploy to server
      uses: appleboy/ssh-action@v0.1.4
      with:
        host: ${{ secrets.HOST }}
        username: ${{ secrets.USERNAME }}
        key: ${{ secrets.KEY }}
        script: |
          cd /var/www/ajcaremotors
          git pull origin main
          composer install --optimize-autoloader --no-dev
          npm install
          npm run production
          php artisan migrate --force
          php artisan config:cache
          php artisan route:cache
          php artisan view:cache
          sudo supervisorctl restart ajcaremotors-worker:*
```

## Troubleshooting

### Common Issues

#### Permission Errors
```bash
# Fix Laravel permissions
sudo chown -R www-data:www-data /var/www/ajcaremotors
sudo chmod -R 755 /var/www/ajcaremotors
sudo chmod -R 775 /var/www/ajcaremotors/storage
sudo chmod -R 775 /var/www/ajcaremotors/bootstrap/cache
```

#### 500 Internal Server Error
1. Check Nginx error logs: `sudo tail -f /var/log/nginx/error.log`
2. Check PHP-FPM logs: `sudo tail -f /var/log/php8.1-fpm.log`
3. Check Laravel logs: `tail -f /var/www/ajcaremotors/storage/logs/laravel.log`

#### Database Connection Issues
```bash
# Test database connection
php artisan tinker
>>> DB::connection()->getPdo();
```

### Useful Commands

```bash
# Check service status
sudo systemctl status nginx php8.1-fpm mysql redis-server

# View logs
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
tail -f /var/www/ajcaremotors/storage/logs/laravel.log

# Clear Laravel caches
php artisan cache:clear
php artisan config:clear
php artisan route:clear
php artisan view:clear

# Check disk space
df -h

# Check memory usage
free -h

# Check running processes
htop
```

## Security Hardening

### Server Security
```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Configure firewall
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https
sudo ufw enable

# Disable root login
sudo sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```

### Application Security
- Regular security updates
- Input validation
- SQL injection prevention
- XSS protection
- CSRF protection
- Rate limiting
- Secure headers

This deployment guide should be customized based on your specific infrastructure and requirements.
