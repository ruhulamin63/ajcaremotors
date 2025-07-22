# Contributing Guidelines

Thank you for your interest in contributing to the AJ Care Motors project! This document provides guidelines and information for contributors.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Development Workflow](#development-workflow)
- [Coding Standards](#coding-standards)
- [Testing Guidelines](#testing-guidelines)
- [Pull Request Process](#pull-request-process)
- [Issue Reporting](#issue-reporting)
- [Documentation](#documentation)
- [Community](#community)

## Code of Conduct

### Our Pledge

We are committed to making participation in this project a harassment-free experience for everyone, regardless of age, body size, disability, ethnicity, gender identity and expression, level of experience, nationality, personal appearance, race, religion, or sexual identity and orientation.

### Our Standards

Examples of behavior that contributes to creating a positive environment include:

- Using welcoming and inclusive language
- Being respectful of differing viewpoints and experiences
- Gracefully accepting constructive criticism
- Focusing on what is best for the community
- Showing empathy towards other community members

### Enforcement

Instances of abusive, harassing, or otherwise unacceptable behavior may be reported by contacting the project team. All complaints will be reviewed and investigated and will result in a response that is deemed necessary and appropriate to the circumstances.

## Getting Started

### Prerequisites

Before contributing, ensure you have:

- PHP 8.0+ installed
- Composer installed
- Node.js 14+ and NPM installed
- MySQL or PostgreSQL database
- Git for version control
- Code editor (VS Code recommended)

### Development Setup

1. **Fork the Repository**
   ```bash
   # Fork the repository on GitHub, then clone your fork
   git clone https://github.com/YOUR_USERNAME/ajcaremotors.git
   cd ajcaremotors
   ```

2. **Set Up Development Environment**
   ```bash
   # Install PHP dependencies
   composer install
   
   # Install Node.js dependencies
   npm install
   
   # Copy environment file
   cp .env.example .env
   
   # Generate application key
   php artisan key:generate
   
   # Set up database
   php artisan migrate
   php artisan db:seed
   ```

3. **Configure Development Tools**
   ```bash
   # Install development dependencies
   composer require --dev laravel/telescope
   php artisan telescope:install
   php artisan migrate
   
   # Install debugging tools
   composer require --dev barryvdh/laravel-debugbar
   ```

### Branch Strategy

We use Git Flow branching strategy:

- **main**: Production-ready code
- **develop**: Integration branch for features
- **feature/**: New features (`feature/bike-management`)
- **bugfix/**: Bug fixes (`bugfix/login-issue`)
- **hotfix/**: Critical production fixes (`hotfix/security-patch`)
- **release/**: Release preparation (`release/v1.1.0`)

## Development Workflow

### 1. Create a Feature Branch

```bash
# Start from develop branch
git checkout develop
git pull origin develop

# Create feature branch
git checkout -b feature/your-feature-name
```

### 2. Make Your Changes

- Follow coding standards (see below)
- Write tests for new functionality
- Update documentation as needed
- Commit frequently with descriptive messages

### 3. Commit Guidelines

#### Commit Message Format
```
type(scope): brief description

Detailed explanation of the change (if needed)

Fixes #issue_number
```

#### Commit Types
- **feat**: New feature
- **fix**: Bug fix
- **docs**: Documentation changes
- **style**: Code style changes (formatting, etc.)
- **refactor**: Code refactoring
- **test**: Adding or updating tests
- **chore**: Maintenance tasks

#### Examples
```bash
git commit -m "feat(auth): add two-factor authentication"
git commit -m "fix(bike): resolve registration validation issue"
git commit -m "docs(api): update endpoint documentation"
```

### 4. Testing

```bash
# Run all tests
php artisan test

# Run specific test suite
php artisan test --testsuite=Feature
php artisan test --testsuite=Unit

# Run with coverage
php artisan test --coverage

# Frontend tests
npm test
```

### 5. Push and Create Pull Request

```bash
# Push your branch
git push origin feature/your-feature-name

# Create pull request on GitHub
```

## Coding Standards

### PHP Standards

We follow **PSR-12** coding standards with additional Laravel conventions:

#### General Rules
- Use 4 spaces for indentation
- Line length should not exceed 120 characters
- Use meaningful variable and function names
- Add type hints for parameters and return types
- Use strict typing when possible

#### Laravel Conventions
```php
<?php

declare(strict_types=1);

namespace App\Http\Controllers;

use App\Models\Bike;
use App\Http\Requests\StoreBikeRequest;
use Illuminate\Http\JsonResponse;

class BikeController extends Controller
{
    public function store(StoreBikeRequest $request): JsonResponse
    {
        $bike = Bike::create($request->validated());
        
        return response()->json([
            'success' => true,
            'data' => $bike,
            'message' => 'Bike created successfully'
        ], 201);
    }
}
```

#### Naming Conventions
- **Classes**: PascalCase (`BikeController`, `ServiceRequest`)
- **Methods**: camelCase (`createBike`, `updateService`)
- **Variables**: camelCase (`$bikeData`, `$serviceRecord`)
- **Constants**: SCREAMING_SNAKE_CASE (`MAX_UPLOAD_SIZE`)
- **Database tables**: snake_case (`bike_services`, `user_policies`)
- **Model attributes**: snake_case (`created_at`, `service_date`)

### JavaScript Standards

#### ES6+ Features
```javascript
// Use arrow functions for short functions
const users = data.map(user => user.name);

// Use destructuring
const { name, email } = user;

// Use template literals
const message = `Welcome ${name}!`;

// Use async/await instead of promises
async function fetchBikes() {
    try {
        const response = await axios.get('/api/bikes');
        return response.data;
    } catch (error) {
        console.error('Error fetching bikes:', error);
    }
}
```

#### Vue.js Components (if applicable)
```vue
<template>
    <div class="bike-list">
        <bike-card 
            v-for="bike in bikes" 
            :key="bike.id" 
            :bike="bike"
            @updated="handleBikeUpdate"
        />
    </div>
</template>

<script>
export default {
    name: 'BikeList',
    
    props: {
        initialBikes: {
            type: Array,
            default: () => []
        }
    },
    
    data() {
        return {
            bikes: [...this.initialBikes]
        };
    },
    
    methods: {
        handleBikeUpdate(updatedBike) {
            const index = this.bikes.findIndex(bike => bike.id === updatedBike.id);
            if (index !== -1) {
                this.bikes.splice(index, 1, updatedBike);
            }
        }
    }
};
</script>
```

### Database Standards

#### Migration Files
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('bike_services', function (Blueprint $table) {
            $table->id();
            $table->foreignId('bike_id')->constrained()->onDelete('cascade');
            $table->string('service_type');
            $table->text('description');
            $table->date('service_date');
            $table->decimal('cost', 8, 2);
            $table->string('technician')->nullable();
            $table->json('parts_used')->nullable();
            $table->enum('status', ['pending', 'in_progress', 'completed', 'cancelled'])
                  ->default('pending');
            $table->timestamps();
            
            $table->index(['bike_id', 'service_date']);
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('bike_services');
    }
};
```

#### Model Definitions
```php
<?php

declare(strict_types=1);

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Bike extends Model
{
    use HasFactory;

    protected $fillable = [
        'make',
        'model',
        'year',
        'license_plate',
        'vin',
        'color',
        'engine_size',
        'owner_id'
    ];

    protected $casts = [
        'year' => 'integer',
        'created_at' => 'datetime',
        'updated_at' => 'datetime'
    ];

    public function owner(): BelongsTo
    {
        return $this->belongsTo(User::class, 'owner_id');
    }

    public function services(): HasMany
    {
        return $this->hasMany(BikeService::class);
    }
}
```

## Testing Guidelines

### Test Structure

#### Feature Tests
```php
<?php

declare(strict_types=1);

namespace Tests\Feature;

use App\Models\User;
use App\Models\Bike;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class BikeManagementTest extends TestCase
{
    use RefreshDatabase;

    public function test_user_can_create_bike(): void
    {
        $user = User::factory()->create();
        
        $bikeData = [
            'make' => 'Honda',
            'model' => 'CBR600RR',
            'year' => 2023,
            'license_plate' => 'ABC-123'
        ];

        $response = $this->actingAs($user)
                         ->postJson('/api/bikes', $bikeData);

        $response->assertStatus(201)
                 ->assertJsonStructure([
                     'success',
                     'data' => [
                         'id',
                         'make',
                         'model',
                         'year',
                         'license_plate'
                     ]
                 ]);

        $this->assertDatabaseHas('bikes', $bikeData);
    }

    public function test_bike_creation_requires_authentication(): void
    {
        $response = $this->postJson('/api/bikes', [
            'make' => 'Honda',
            'model' => 'CBR600RR'
        ]);

        $response->assertStatus(401);
    }
}
```

#### Unit Tests
```php
<?php

declare(strict_types=1);

namespace Tests\Unit;

use App\Models\Bike;
use App\Models\User;
use Tests\TestCase;

class BikeTest extends TestCase
{
    public function test_bike_belongs_to_user(): void
    {
        $user = User::factory()->create();
        $bike = Bike::factory()->create(['owner_id' => $user->id]);

        $this->assertInstanceOf(User::class, $bike->owner);
        $this->assertEquals($user->id, $bike->owner->id);
    }

    public function test_bike_can_have_services(): void
    {
        $bike = Bike::factory()->create();
        
        $this->assertInstanceOf(
            \Illuminate\Database\Eloquent\Collection::class,
            $bike->services
        );
    }
}
```

### Test Data

#### Factories
```php
<?php

declare(strict_types=1);

namespace Database\Factories;

use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Factory;

class BikeFactory extends Factory
{
    public function definition(): array
    {
        return [
            'make' => $this->faker->randomElement(['Honda', 'Yamaha', 'Kawasaki', 'Suzuki']),
            'model' => $this->faker->word,
            'year' => $this->faker->numberBetween(2000, 2024),
            'license_plate' => $this->faker->unique()->regexify('[A-Z]{3}-[0-9]{3}'),
            'vin' => $this->faker->unique()->regexify('[A-Z0-9]{17}'),
            'color' => $this->faker->colorName,
            'engine_size' => $this->faker->randomElement(['250cc', '600cc', '1000cc']),
            'owner_id' => User::factory()
        ];
    }
}
```

## Pull Request Process

### Before Submitting

1. **Self Review**
   - [ ] Code follows project standards
   - [ ] All tests pass
   - [ ] Documentation updated
   - [ ] No merge conflicts

2. **Testing Checklist**
   - [ ] Unit tests written for new functions
   - [ ] Feature tests for new endpoints
   - [ ] Manual testing completed
   - [ ] Edge cases considered

3. **Documentation Checklist**
   - [ ] Code comments added where needed
   - [ ] API documentation updated
   - [ ] README updated if needed
   - [ ] Changelog entry added

### Pull Request Template

```markdown
## Description
Brief description of changes made.

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
- [ ] Unit tests pass
- [ ] Feature tests pass
- [ ] Manual testing completed

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Documentation updated
- [ ] No merge conflicts

## Screenshots (if applicable)
Add screenshots of new features or UI changes.

## Related Issues
Fixes #issue_number
```

### Review Process

1. **Automated Checks**
   - Code style validation
   - Test suite execution
   - Security scanning

2. **Peer Review**
   - At least one team member review required
   - Focus on code quality, security, and maintainability
   - Address all feedback before merging

3. **Final Testing**
   - Deploy to staging environment
   - Perform final acceptance testing
   - Approve for production deployment

## Issue Reporting

### Bug Reports

Use the bug report template:

```markdown
**Bug Description**
A clear description of the bug.

**Steps to Reproduce**
1. Go to '...'
2. Click on '....'
3. Scroll down to '....'
4. See error

**Expected Behavior**
What you expected to happen.

**Actual Behavior**
What actually happened.

**Environment**
- OS: [e.g. Windows 10]
- Browser: [e.g. Chrome 96]
- Version: [e.g. 1.0.0]

**Screenshots**
Add screenshots if applicable.

**Additional Context**
Any other context about the problem.
```

### Feature Requests

Use the feature request template:

```markdown
**Feature Description**
A clear description of the feature you'd like to see.

**Problem Statement**
What problem does this feature solve?

**Proposed Solution**
How would you like this feature to work?

**Alternatives Considered**
Other solutions you've considered.

**Additional Context**
Any other context about the feature request.
```

## Documentation

### Code Documentation

#### PHPDoc Standards
```php
/**
 * Create a new bike service record.
 *
 * @param  \App\Http\Requests\StoreBikeServiceRequest  $request
 * @return \Illuminate\Http\JsonResponse
 * 
 * @throws \Illuminate\Validation\ValidationException
 */
public function store(StoreBikeServiceRequest $request): JsonResponse
{
    // Implementation
}
```

#### API Documentation
- Use clear endpoint descriptions
- Include request/response examples
- Document all parameters
- Specify error codes and messages

### README Updates

When adding new features:
- Update installation instructions if needed
- Add configuration steps
- Include usage examples
- Update the feature list

## Community

### Communication Channels

- **GitHub Issues**: Bug reports and feature requests
- **GitHub Discussions**: General questions and ideas
- **Email**: Direct contact for sensitive issues

### Getting Help

1. Check existing documentation
2. Search existing issues
3. Ask in GitHub Discussions
4. Create a new issue if needed

### Recognition

Contributors will be:
- Added to the contributors list
- Mentioned in release notes
- Invited to join the core team (for significant contributions)

## Development Tools

### Recommended Extensions (VS Code)

- PHP Intelephense
- Laravel Extension Pack
- GitLens
- ESLint
- Prettier
- Thunder Client (API testing)

### Useful Commands

```bash
# Code style checking
./vendor/bin/phpcs --standard=PSR12 app/

# Code style fixing
./vendor/bin/phpcbf --standard=PSR12 app/

# Static analysis
./vendor/bin/phpstan analyse

# Security checking
composer audit
```

Thank you for contributing to AJ Care Motors! 🚗🔧
