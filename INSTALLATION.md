# Installation Guide

This guide provides detailed installation instructions for the AJ Care Motors platform.

## Prerequisites

Before installing, ensure you have the following software installed on your system:

### Required Software
- **PHP**: Version 7.3 or higher (8.0+ recommended)
- **Composer**: Latest version
- **Node.js**: Version 14.x or higher
- **NPM**: Comes with Node.js
- **MySQL**: Version 5.7 or higher (or PostgreSQL 9.6+)
- **Git**: For version control

### PHP Extensions
Ensure the following PHP extensions are installed:
- BCMath
- Ctype
- Fileinfo
- JSON
- Mbstring
- OpenSSL
- PDO
- Tokenizer
- XML
- cURL

## Step-by-Step Installation

### 1. Environment Setup

#### Windows (using Laragon/XAMPP)
```powershell
# Check PHP version
php -v

# Check Composer installation
composer --version

# Check Node.js and NPM
node -v
npm -v
```

#### Linux/macOS
```bash
# Install PHP (Ubuntu/Debian)
sudo apt update
sudo apt install php8.0 php8.0-cli php8.0-common php8.0-mysql php8.0-xml php8.0-curl php8.0-mbstring

# Install Composer
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer

# Install Node.js (using nvm)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
nvm install 16
nvm use 16
```

### 2. Project Setup

#### Clone Repository
```bash
git clone https://github.com/ruhulamin63/ajcaremotors.git
cd ajcaremotors
```

#### Install Dependencies
```bash
# Install PHP dependencies
composer install

# Install Node.js dependencies
npm install
```

### 3. Environment Configuration

#### Copy Environment File
```bash
# Windows PowerShell
Copy-Item .env.example .env

# Linux/macOS
cp .env.example .env
```

#### Generate Application Key
```bash
php artisan key:generate
```

#### Configure Environment Variables
Edit the `.env` file with your settings:

```env
APP_NAME="AJ Care Motors"
APP_ENV=local
APP_KEY=base64:your_generated_key_here
APP_DEBUG=true
APP_URL=http://localhost:8000

LOG_CHANNEL=stack
LOG_LEVEL=debug

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=ajcaremotors
DB_USERNAME=root
DB_PASSWORD=your_password

BROADCAST_DRIVER=log
CACHE_DRIVER=file
FILESYSTEM_DRIVER=local
QUEUE_CONNECTION=sync
SESSION_DRIVER=file
SESSION_LIFETIME=120

MAIL_MAILER=smtp
MAIL_HOST=localhost
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS=noreply@ajcaremotors.com
MAIL_FROM_NAME="${APP_NAME}"
```

### 4. Database Setup

#### Create Database
```sql
-- MySQL
CREATE DATABASE ajcaremotors CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- PostgreSQL
CREATE DATABASE ajcaremotors;
```

#### Run Migrations
```bash
# Run database migrations
php artisan migrate

# (Optional) Seed with sample data
php artisan db:seed
```

### 5. File Permissions

#### Set Proper Permissions (Linux/macOS)
```bash
sudo chown -R www-data:www-data storage bootstrap/cache
sudo chmod -R 775 storage bootstrap/cache
```

#### Windows Permissions
Ensure the following directories are writable:
- `storage/`
- `bootstrap/cache/`

### 6. Build Frontend Assets

```bash
# Development build
npm run dev

# Production build (for live sites)
npm run prod
```

### 7. Start Development Server

```bash
php artisan serve
```

Your application will be available at `http://localhost:8000`

## Advanced Installation Options

### Using Docker
```bash
# If you prefer Docker (requires Docker Desktop)
# Create docker-compose.yml and run:
docker-compose up -d
```

### Using Laravel Sail
```bash
# Install Laravel Sail
composer require laravel/sail --dev
php artisan sail:install
./vendor/bin/sail up
```

## Verification

### Check Installation
```bash
# Verify application status
php artisan about

# Check if all services are running
php artisan tinker
```

### Test Basic Functionality
1. Visit `http://localhost:8000`
2. You should see the Laravel welcome page
3. Check that assets are loading correctly

## Troubleshooting

### Common Installation Issues

#### Permission Denied Errors
```bash
# Fix storage permissions
sudo chmod -R 775 storage
sudo chmod -R 775 bootstrap/cache
```

#### Composer Memory Limit
```bash
# Increase PHP memory limit
php -d memory_limit=-1 /usr/local/bin/composer install
```

#### Node.js/NPM Issues
```bash
# Clear npm cache
npm cache clean --force

# Delete node_modules and reinstall
rm -rf node_modules package-lock.json
npm install
```

#### Database Connection Issues
1. Verify database credentials in `.env`
2. Ensure database server is running
3. Test connection:
```bash
php artisan tinker
DB::connection()->getPdo();
```

#### Asset Compilation Issues
```bash
# Clear compiled assets
rm -rf public/css/* public/js/*

# Rebuild assets
npm run dev
```

### Getting Help

If you encounter issues not covered here:
1. Check the main [README.md](README.md)
2. Review Laravel documentation
3. Check the project's issue tracker
4. Contact the development team

## Next Steps

After successful installation:
1. Review the [Configuration Guide](CONFIGURATION.md)
2. Set up your development environment
3. Read the [API Documentation](API.md)
4. Start developing your features

## Production Deployment

For production deployment, see the [Deployment Guide](DEPLOYMENT.md).
