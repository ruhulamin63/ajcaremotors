# AJ Care Motors - Bike Services Platform

A comprehensive bike services management platform built with Laravel 8.x. This application provides policy management and service tracking for motorcycle maintenance and repair services.

## Table of Contents

- [Project Overview](#project-overview)
- [Tech Stack](#tech-stack)
- [System Requirements](#system-requirements)
- [Installation](#installation)
- [Configuration](#configuration)
- [Project Structure](#project-structure)
- [Development Workflow](#development-workflow)
- [Testing](#testing)
- [Deployment](#deployment)
- [API Documentation](#api-documentation)
- [Contributing](#contributing)

## Project Overview

AJ Care Motors is a bike services policy management system designed to handle:
- Motorcycle service scheduling
- Maintenance tracking
- Customer management
- Service policy administration
- Repair history documentation

## Tech Stack

### Backend
- **Framework**: Laravel 8.x
- **PHP Version**: ^7.3|^8.0
- **Authentication**: Laravel Sanctum
- **Database**: MySQL (configurable)

### Frontend Assets
- **Build Tool**: Laravel Mix 6.x
- **JavaScript**: Vanilla JS with Axios
- **CSS**: Basic styling with app.css
- **Utilities**: Lodash

### Development Tools
- **Testing**: PHPUnit 9.x
- **Code Quality**: Laravel Pint (via Collision)
- **Package Manager**: Composer & NPM
- **Local Development**: Laravel Artisan Server

## System Requirements

- PHP >= 7.3 or 8.0
- Composer
- Node.js & NPM
- MySQL 5.7+ or PostgreSQL 9.6+
- Web server (Apache/Nginx) for production

## Installation

### 1. Clone the Repository
```bash
git clone https://github.com/ruhulamin63/AJ-Care-Motors-Bike-Services-Platform.git
cd ajcaremotors
```

### 2. Install PHP Dependencies
```bash
composer install
```

### 3. Install Node Dependencies
```bash
npm install
```

### 4. Environment Setup
```bash
# Copy environment file
cp .env.example .env

# Generate application key
php artisan key:generate
```

### 5. Database Setup
```bash
# Create database and run migrations
php artisan migrate

# (Optional) Seed database with sample data
php artisan db:seed
```

### 6. Build Frontend Assets
```bash
# For development
npm run dev

# For production
npm run prod

# For watching changes during development
npm run watch
```

## Configuration

### Environment Variables

Edit the `.env` file with your specific configuration:

```env
# Application
APP_NAME="AJ Care Motors"
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost

# Database
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=ajcaremotors
DB_USERNAME=your_username
DB_PASSWORD=your_password

# Mail Configuration
MAIL_MAILER=smtp
MAIL_HOST=your_smtp_host
MAIL_PORT=587
MAIL_USERNAME=your_email
MAIL_PASSWORD=your_password
MAIL_ENCRYPTION=tls
```

### Key Configuration Files

- `config/app.php` - Application settings
- `config/database.php` - Database configuration
- `config/auth.php` - Authentication settings
- `config/sanctum.php` - API authentication
- `config/cors.php` - CORS settings for API

## Project Structure

```
ajcaremotors/
├── app/
│   ├── Console/           # Artisan commands
│   ├── Exceptions/        # Exception handlers
│   ├── Http/
│   │   ├── Controllers/   # Application controllers
│   │   ├── Middleware/    # HTTP middleware
│   │   └── Kernel.php     # HTTP kernel
│   ├── Models/            # Eloquent models
│   └── Providers/         # Service providers
├── bootstrap/             # Framework bootstrap files
├── config/               # Configuration files
├── database/
│   ├── factories/        # Model factories
│   ├── migrations/       # Database migrations
│   └── seeders/          # Database seeders
├── public/               # Web server document root
├── resources/
│   ├── css/              # Stylesheet files
│   ├── js/               # JavaScript files
│   ├── lang/             # Language files
│   └── views/            # Blade templates
├── routes/
│   ├── web.php           # Web routes
│   ├── api.php           # API routes
│   ├── console.php       # Console routes
│   └── channels.php      # Broadcast channels
├── storage/              # Application storage
├── tests/                # Application tests
└── vendor/               # Composer dependencies
```

## Development Workflow

### Starting the Development Server
```bash
php artisan serve
```
Application will be available at `http://localhost:8000`

### Asset Compilation
```bash
# Development build
npm run dev

# Watch for changes
npm run watch

# Hot module replacement
npm run hot

# Production build
npm run prod
```

### Database Operations
```bash
# Create new migration
php artisan make:migration create_table_name

# Run migrations
php artisan migrate

# Rollback migrations
php artisan migrate:rollback

# Create model with migration
php artisan make:model ModelName -m

# Create seeder
php artisan make:seeder TableSeeder
```

### Generating Components
```bash
# Create controller
php artisan make:controller ControllerName

# Create model
php artisan make:model ModelName

# Create middleware
php artisan make:middleware MiddlewareName

# Create request
php artisan make:request RequestName
```

## Testing

### Running Tests
```bash
# Run all tests
php artisan test

# Run specific test file
php artisan test tests/Feature/ExampleTest.php

# Run with coverage
php artisan test --coverage
```

### Test Structure
- `tests/Feature/` - Feature tests (HTTP tests)
- `tests/Unit/` - Unit tests (isolated component tests)

## API Documentation

### Authentication
The API uses Laravel Sanctum for authentication. Include the token in the Authorization header:
```
Authorization: Bearer {your-token}
```

### Endpoints
- `GET /api/user` - Get authenticated user information

### CORS Configuration
CORS is configured in `config/cors.php` and enabled via the `fruitcake/laravel-cors` package.

## Deployment

### Production Checklist
1. Set `APP_ENV=production` in `.env`
2. Set `APP_DEBUG=false` in `.env`
3. Run `composer install --optimize-autoloader --no-dev`
4. Run `npm run prod`
5. Run `php artisan config:cache`
6. Run `php artisan route:cache`
7. Run `php artisan view:cache`
8. Set proper file permissions
9. Configure web server (Apache/Nginx)

### Server Configuration
Ensure your web server points to the `public/` directory and has proper rewrite rules for Laravel.

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Code Style
- Follow PSR-12 coding standards
- Use Laravel naming conventions
- Write descriptive commit messages
- Include tests for new features

## Troubleshooting

### Common Issues

1. **Permission Errors**: Ensure `storage/` and `bootstrap/cache/` are writable
2. **Key Not Set**: Run `php artisan key:generate`
3. **Database Connection**: Check your `.env` database credentials
4. **Asset Not Found**: Run `npm run dev` to compile assets

### Useful Commands
```bash
# Clear all caches
php artisan optimize:clear

# View routes
php artisan route:list

# Check application status
php artisan about

# View logs
tail -f storage/logs/laravel.log
```

## License

This project is open-sourced software licensed under the [MIT license](https://opensource.org/licenses/MIT).

## Support

For support and questions, please contact the development team or create an issue in the repository.
