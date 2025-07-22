# API Documentation

This document provides comprehensive API documentation for the AJ Care Motors platform.

## Table of Contents

- [Authentication](#authentication)
- [Base URL](#base-url)
- [Response Format](#response-format)
- [Error Handling](#error-handling)
- [Rate Limiting](#rate-limiting)
- [Endpoints](#endpoints)
- [Request Examples](#request-examples)
- [SDK Examples](#sdk-examples)

## Authentication

The AJ Care Motors API uses Laravel Sanctum for authentication. You need to include an API token in the Authorization header for authenticated requests.

### Getting an API Token

1. **Register/Login** to get a session
2. **Generate Token** via the dashboard or API endpoint
3. **Include Token** in subsequent requests

### Authentication Header

```http
Authorization: Bearer {your-api-token}
```

### Token Management

```http
POST /api/tokens/create
Content-Type: application/json

{
    "name": "Mobile App Token",
    "abilities": ["read", "write"]
}
```

## Base URL

```
Development: http://localhost:8000/api
Production: https://your-domain.com/api
```

## Response Format

All API responses follow a consistent JSON format:

### Success Response
```json
{
    "success": true,
    "data": {
        // Response data
    },
    "message": "Operation completed successfully",
    "meta": {
        "timestamp": "2025-07-22T10:30:00Z",
        "version": "1.0"
    }
}
```

### Error Response
```json
{
    "success": false,
    "error": {
        "code": "VALIDATION_ERROR",
        "message": "The given data was invalid.",
        "details": {
            "email": ["The email field is required."]
        }
    },
    "meta": {
        "timestamp": "2025-07-22T10:30:00Z",
        "version": "1.0"
    }
}
```

## Error Handling

### HTTP Status Codes

| Code | Description |
|------|-------------|
| 200 | OK - Request successful |
| 201 | Created - Resource created successfully |
| 400 | Bad Request - Invalid request data |
| 401 | Unauthorized - Authentication required |
| 403 | Forbidden - Access denied |
| 404 | Not Found - Resource not found |
| 422 | Unprocessable Entity - Validation errors |
| 429 | Too Many Requests - Rate limit exceeded |
| 500 | Internal Server Error - Server error |

### Error Codes

| Code | Description |
|------|-------------|
| `AUTHENTICATION_REQUIRED` | User authentication required |
| `INVALID_TOKEN` | Invalid or expired token |
| `VALIDATION_ERROR` | Request validation failed |
| `RESOURCE_NOT_FOUND` | Requested resource not found |
| `PERMISSION_DENIED` | Insufficient permissions |
| `RATE_LIMIT_EXCEEDED` | API rate limit exceeded |

## Rate Limiting

- **Default Limit**: 60 requests per minute per IP
- **Authenticated Limit**: 120 requests per minute per user
- **Headers**: Rate limit information is included in response headers

```http
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 59
X-RateLimit-Reset: 1642780800
```

## Endpoints

### Authentication Endpoints

#### Get Authenticated User
```http
GET /api/user
Authorization: Bearer {token}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "id": 1,
        "name": "John Doe",
        "email": "john@example.com",
        "email_verified_at": "2025-07-22T10:30:00Z",
        "created_at": "2025-07-22T10:30:00Z",
        "updated_at": "2025-07-22T10:30:00Z"
    }
}
```

### User Management Endpoints

#### Register User
```http
POST /api/auth/register
Content-Type: application/json

{
    "name": "John Doe",
    "email": "john@example.com",
    "password": "password123",
    "password_confirmation": "password123"
}
```

#### Login User
```http
POST /api/auth/login
Content-Type: application/json

{
    "email": "john@example.com",
    "password": "password123"
}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "user": {
            "id": 1,
            "name": "John Doe",
            "email": "john@example.com"
        },
        "token": "1|abc123def456ghi789..."
    }
}
```

#### Logout User
```http
POST /api/auth/logout
Authorization: Bearer {token}
```

### Bike Management Endpoints

#### List Bikes
```http
GET /api/bikes
Authorization: Bearer {token}
```

**Query Parameters:**
- `page` (int): Page number (default: 1)
- `per_page` (int): Items per page (default: 15, max: 100)
- `search` (string): Search term
- `status` (string): Filter by status
- `sort` (string): Sort field (default: created_at)
- `order` (string): Sort order (asc, desc)

**Response:**
```json
{
    "success": true,
    "data": {
        "bikes": [
            {
                "id": 1,
                "make": "Honda",
                "model": "CBR600RR",
                "year": 2023,
                "license_plate": "ABC-123",
                "owner_id": 1,
                "status": "active",
                "created_at": "2025-07-22T10:30:00Z",
                "updated_at": "2025-07-22T10:30:00Z"
            }
        ],
        "pagination": {
            "current_page": 1,
            "total_pages": 5,
            "total_items": 75,
            "per_page": 15
        }
    }
}
```

#### Get Single Bike
```http
GET /api/bikes/{id}
Authorization: Bearer {token}
```

#### Create Bike
```http
POST /api/bikes
Authorization: Bearer {token}
Content-Type: application/json

{
    "make": "Honda",
    "model": "CBR600RR",
    "year": 2023,
    "license_plate": "ABC-123",
    "vin": "1HGBH41JXMN109186",
    "color": "Red",
    "engine_size": "600cc"
}
```

#### Update Bike
```http
PUT /api/bikes/{id}
Authorization: Bearer {token}
Content-Type: application/json

{
    "make": "Honda",
    "model": "CBR600RR",
    "year": 2023,
    "status": "maintenance"
}
```

#### Delete Bike
```http
DELETE /api/bikes/{id}
Authorization: Bearer {token}
```

### Service Management Endpoints

#### List Services
```http
GET /api/services
Authorization: Bearer {token}
```

#### Create Service Record
```http
POST /api/services
Authorization: Bearer {token}
Content-Type: application/json

{
    "bike_id": 1,
    "service_type": "maintenance",
    "description": "Regular oil change and inspection",
    "service_date": "2025-07-22",
    "cost": 75.00,
    "technician": "Mike Johnson",
    "parts_used": [
        {
            "name": "Engine Oil",
            "quantity": 1,
            "cost": 25.00
        }
    ]
}
```

#### Get Service History
```http
GET /api/bikes/{bike_id}/services
Authorization: Bearer {token}
```

### Policy Management Endpoints

#### List Policies
```http
GET /api/policies
Authorization: Bearer {token}
```

#### Create Policy
```http
POST /api/policies
Authorization: Bearer {token}
Content-Type: application/json

{
    "name": "Basic Maintenance Plan",
    "description": "Covers regular maintenance services",
    "coverage_type": "maintenance",
    "duration_months": 12,
    "price": 299.99,
    "terms": "Standard terms and conditions apply"
}
```

### Appointment Endpoints

#### List Appointments
```http
GET /api/appointments
Authorization: Bearer {token}
```

#### Book Appointment
```http
POST /api/appointments
Authorization: Bearer {token}
Content-Type: application/json

{
    "bike_id": 1,
    "service_type": "maintenance",
    "preferred_date": "2025-07-25",
    "preferred_time": "10:00",
    "notes": "Oil change needed"
}
```

#### Update Appointment
```http
PUT /api/appointments/{id}
Authorization: Bearer {token}
Content-Type: application/json

{
    "status": "confirmed",
    "scheduled_date": "2025-07-25",
    "scheduled_time": "10:00"
}
```

## Request Examples

### cURL Examples

#### Get User Profile
```bash
curl -X GET "http://localhost:8000/api/user" \
     -H "Authorization: Bearer your-token-here" \
     -H "Accept: application/json"
```

#### Create New Bike
```bash
curl -X POST "http://localhost:8000/api/bikes" \
     -H "Authorization: Bearer your-token-here" \
     -H "Content-Type: application/json" \
     -H "Accept: application/json" \
     -d '{
         "make": "Honda",
         "model": "CBR600RR",
         "year": 2023,
         "license_plate": "ABC-123"
     }'
```

### JavaScript Examples

#### Using Axios
```javascript
// Set default headers
axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
axios.defaults.headers.common['Accept'] = 'application/json';

// Get user profile
const getUserProfile = async () => {
    try {
        const response = await axios.get('/api/user');
        return response.data;
    } catch (error) {
        console.error('Error fetching user profile:', error.response.data);
    }
};

// Create new bike
const createBike = async (bikeData) => {
    try {
        const response = await axios.post('/api/bikes', bikeData);
        return response.data;
    } catch (error) {
        console.error('Error creating bike:', error.response.data);
    }
};
```

#### Using Fetch API
```javascript
// Helper function for API calls
const apiCall = async (url, options = {}) => {
    const defaultOptions = {
        headers: {
            'Authorization': `Bearer ${token}`,
            'Accept': 'application/json',
            'Content-Type': 'application/json'
        }
    };

    const response = await fetch(`/api${url}`, {
        ...defaultOptions,
        ...options,
        headers: { ...defaultOptions.headers, ...options.headers }
    });

    if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
    }

    return response.json();
};

// Usage examples
const user = await apiCall('/user');
const bikes = await apiCall('/bikes?page=1&per_page=10');
```

### PHP Examples

#### Using Guzzle HTTP Client
```php
use GuzzleHttp\Client;

$client = new Client([
    'base_uri' => 'http://localhost:8000/api/',
    'headers' => [
        'Authorization' => 'Bearer ' . $token,
        'Accept' => 'application/json'
    ]
]);

// Get user profile
$response = $client->get('user');
$user = json_decode($response->getBody(), true);

// Create new bike
$response = $client->post('bikes', [
    'json' => [
        'make' => 'Honda',
        'model' => 'CBR600RR',
        'year' => 2023,
        'license_plate' => 'ABC-123'
    ]
]);
```

## SDK Examples

### Laravel HTTP Client
```php
use Illuminate\Support\Facades\Http;

// Configure base client
$client = Http::withHeaders([
    'Authorization' => 'Bearer ' . $token,
    'Accept' => 'application/json'
])->baseUrl('http://localhost:8000/api/');

// Get user
$response = $client->get('user');
$user = $response->json();

// Create bike
$response = $client->post('bikes', [
    'make' => 'Honda',
    'model' => 'CBR600RR',
    'year' => 2023,
    'license_plate' => 'ABC-123'
]);
```

## Webhooks

### Setting Up Webhooks
```http
POST /api/webhooks
Authorization: Bearer {token}
Content-Type: application/json

{
    "url": "https://your-app.com/webhooks/ajcaremotors",
    "events": ["service.completed", "appointment.scheduled"],
    "secret": "your-webhook-secret"
}
```

### Webhook Events
- `service.completed` - Service work completed
- `appointment.scheduled` - New appointment scheduled
- `appointment.cancelled` - Appointment cancelled
- `bike.registered` - New bike registered
- `policy.activated` - Policy activated

### Webhook Payload Example
```json
{
    "event": "service.completed",
    "data": {
        "service_id": 123,
        "bike_id": 45,
        "completion_date": "2025-07-22T15:30:00Z"
    },
    "timestamp": "2025-07-22T15:30:00Z",
    "signature": "sha256=abc123..."
}
```

## Testing

### Test Environment
```
Base URL: http://localhost:8000/api
Test Token: Use the provided test token for development
```

### Postman Collection
A Postman collection is available for testing all endpoints. Import the collection file from the repository.

### API Testing Tools
- **Postman**: For manual API testing
- **Insomnia**: Alternative REST client
- **phpunit**: For automated testing

## Versioning

- Current Version: v1
- Version Header: `Accept: application/vnd.ajcaremotors.v1+json`
- Backward Compatibility: Maintained for major versions

## Support

For API support:
- Check the error response for detailed information
- Review this documentation
- Contact the development team
- Check the project's issue tracker

## Changelog

### v1.0.0 (Current)
- Initial API release
- User authentication
- Basic CRUD operations
- Service management
- Policy management
