# EMRims SO Suite - REST API Documentation

## Overview

The EMRims SO Suite API is a RESTful service providing endpoints for:
- User authentication and authorization
- Lead management and CRM operations
- Measurement tracking and EMR sizing
- Quote generation and tracking
- Activity logging

## Base URL

```
http://localhost:5000/api  (development)
https://api.emrims.io      (production)
```

## Authentication

### Login
**Endpoint**: `POST /auth/login`

**Request**:
```json
{
  "username": "admin",
  "password": "Admin"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIs...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIs...",
    "user": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "username": "admin",
      "email": "admin@emrims.io",
      "full_name": "Administrator",
      "role": "admin"
    }
  }
}
```

**Error** (401 Unauthorized):
```json
{
  "success": false,
  "error": "Invalid credentials"
}
```

---

### Validate Token
**Endpoint**: `GET /auth/validate`

**Headers**:
```
Authorization: Bearer {access_token}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "valid": true,
    "user_id": "550e8400-e29b-41d4-a716-446655440000",
    "expires_in": 3600
  }
}
```

---

### Refresh Token
**Endpoint**: `POST /auth/refresh`

**Request**:
```json
{
  "refresh_token": "eyJhbGciOiJIUzI1NiIs..."
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIs..."
  }
}
```

---

### Logout
**Endpoint**: `POST /auth/logout`

**Headers**:
```
Authorization: Bearer {access_token}
```

**Response** (200 OK):
```json
{
  "success": true,
  "message": "Logged out successfully"
}
```

---

## Leads

### List Leads
**Endpoint**: `GET /leads`

**Query Parameters**:
- `page` (default: 1)
- `limit` (default: 20, max: 100)
- `status` (optional: new, contacted, quoted, converted, lost)
- `search` (optional: search by name/email/phone)

**Headers**:
```
Authorization: Bearer {access_token}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "user_id": "550e8400-e29b-41d4-a716-446655440001",
      "first_name": "John",
      "last_name": "Doe",
      "email": "john@example.com",
      "phone": "+2348012345678",
      "company": "Medical Center X",
      "position": "Hospital Manager",
      "status": "contacted",
      "source": "referral",
      "notes": "Interested in bulk order",
      "last_contacted": "2026-04-20T10:30:00Z",
      "created_at": "2026-04-15T08:00:00Z",
      "updated_at": "2026-04-20T10:30:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "pages": 8
  }
}
```

---

### Get Single Lead
**Endpoint**: `GET /leads/:id`

**Headers**:
```
Authorization: Bearer {access_token}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "user_id": "550e8400-e29b-41d4-a716-446655440001",
    "first_name": "John",
    "last_name": "Doe",
    "email": "john@example.com",
    "phone": "+2348012345678",
    "company": "Medical Center X",
    "position": "Hospital Manager",
    "status": "contacted",
    "source": "referral",
    "notes": "Interested in bulk order",
    "last_contacted": "2026-04-20T10:30:00Z",
    "created_at": "2026-04-15T08:00:00Z",
    "updated_at": "2026-04-20T10:30:00Z"
  }
}
```

---

### Create Lead
**Endpoint**: `POST /leads`

**Headers**:
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

**Request**:
```json
{
  "first_name": "Jane",
  "last_name": "Smith",
  "email": "jane@example.com",
  "phone": "+2348098765432",
  "company": "Clinic Y",
  "position": "Director",
  "source": "website"
}
```

**Response** (201 Created):
```json
{
  "success": true,
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440002",
    "user_id": "550e8400-e29b-41d4-a716-446655440001",
    "first_name": "Jane",
    "last_name": "Smith",
    "email": "jane@example.com",
    "phone": "+2348098765432",
    "company": "Clinic Y",
    "position": "Director",
    "status": "new",
    "source": "website",
    "created_at": "2026-04-26T12:00:00Z",
    "updated_at": "2026-04-26T12:00:00Z"
  },
  "message": "Lead created successfully"
}
```

---

### Update Lead
**Endpoint**: `PUT /leads/:id`

**Headers**:
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

**Request** (partial update):
```json
{
  "status": "quoted",
  "notes": "Updated with new information"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "status": "quoted",
    "notes": "Updated with new information",
    "updated_at": "2026-04-26T12:30:00Z"
  },
  "message": "Lead updated successfully"
}
```

---

### Delete Lead
**Endpoint**: `DELETE /leads/:id`

**Headers**:
```
Authorization: Bearer {access_token}
```

**Response** (200 OK):
```json
{
  "success": true,
  "message": "Lead deleted successfully"
}
```

---

## Measurements

### Create Measurement
**Endpoint**: `POST /measurements`

**Headers**:
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

**Request**:
```json
{
  "lead_id": "550e8400-e29b-41d4-a716-446655440000",
  "measurement_type": "manual",
  "data": {
    "chest": 38.5,
    "waist": 32.0,
    "length": 28.5,
    "sleeve_length": 24.0,
    "shoulder_width": 17.5,
    "unit": "inches"
  },
  "fabric_tier": "Regal Signature"
}
```

**Response** (201 Created):
```json
{
  "success": true,
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440010",
    "lead_id": "550e8400-e29b-41d4-a716-446655440000",
    "measurement_type": "manual",
    "data": {
      "chest": 38.5,
      "waist": 32.0,
      "length": 28.5,
      "sleeve_length": 24.0,
      "shoulder_width": 17.5,
      "unit": "inches"
    },
    "recommended_size": "L",
    "fabric_tier": "Regal Signature",
    "created_at": "2026-04-26T12:00:00Z"
  },
  "message": "Measurement recorded successfully"
}
```

---

### Get Measurements for Lead
**Endpoint**: `GET /leads/:lead_id/measurements`

**Headers**:
```
Authorization: Bearer {access_token}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440010",
      "lead_id": "550e8400-e29b-41d4-a716-446655440000",
      "measurement_type": "manual",
      "recommended_size": "L",
      "created_at": "2026-04-26T12:00:00Z"
    }
  ]
}
```

---

## Quotes

### Create Quote
**Endpoint**: `POST /quotes`

**Headers**:
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

**Request**:
```json
{
  "lead_id": "550e8400-e29b-41d4-a716-446655440000",
  "items": [
    {
      "product_name": "Unisex Scrub Set",
      "product_category": "scrubs",
      "size": "L",
      "fabric_tier": "Regal Signature",
      "quantity": 5,
      "unit_price": 32000
    }
  ],
  "tax_amount": 80000,
  "discount_amount": 0,
  "notes": "Bulk order - delivery in 2 weeks",
  "valid_until": "2026-05-26"
}
```

**Response** (201 Created):
```json
{
  "success": true,
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440020",
    "lead_id": "550e8400-e29b-41d4-a716-446655440000",
    "quote_number": "QT-2026-0001",
    "status": "draft",
    "total_amount": 160000,
    "tax_amount": 80000,
    "discount_amount": 0,
    "currency": "NGN",
    "items": [
      {
        "id": "550e8400-e29b-41d4-a716-446655440021",
        "product_name": "Unisex Scrub Set",
        "quantity": 5,
        "unit_price": 32000,
        "total_price": 160000
      }
    ],
    "valid_until": "2026-05-26",
    "created_at": "2026-04-26T12:00:00Z"
  },
  "message": "Quote created successfully"
}
```

---

### Get Quotes for Lead
**Endpoint**: `GET /leads/:lead_id/quotes`

**Headers**:
```
Authorization: Bearer {access_token}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440020",
      "quote_number": "QT-2026-0001",
      "status": "draft",
      "total_amount": 160000,
      "created_at": "2026-04-26T12:00:00Z"
    }
  ]
}
```

---

### Send Quote
**Endpoint**: `POST /quotes/:id/send`

**Headers**:
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

**Request**:
```json
{
  "send_to": "email",
  "message": "Please review the attached quote"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440020",
    "status": "sent",
    "sent_at": "2026-04-26T12:05:00Z"
  },
  "message": "Quote sent to customer"
}
```

---

## Activities

### Get Lead Activity Timeline
**Endpoint**: `GET /leads/:lead_id/activities`

**Headers**:
```
Authorization: Bearer {access_token}
```

**Query Parameters**:
- `limit` (default: 50)

**Response** (200 OK):
```json
{
  "success": true,
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440030",
      "user_id": "550e8400-e29b-41d4-a716-446655440001",
      "activity_type": "measurement",
      "subject": "Body measurements recorded",
      "description": "Recorded manual measurements",
      "created_at": "2026-04-26T12:00:00Z"
    },
    {
      "id": "550e8400-e29b-41d4-a716-446655440031",
      "user_id": "550e8400-e29b-41d4-a716-446655440001",
      "activity_type": "quote",
      "subject": "Quote QT-2026-0001 created",
      "description": "New quote generated for 5 sets",
      "created_at": "2026-04-26T12:05:00Z"
    }
  ]
}
```

---

## Error Responses

### 400 Bad Request
```json
{
  "success": false,
  "error": "Validation failed",
  "details": {
    "email": "Invalid email format",
    "phone": "Phone is required"
  }
}
```

### 401 Unauthorized
```json
{
  "success": false,
  "error": "Token expired or invalid"
}
```

### 403 Forbidden
```json
{
  "success": false,
  "error": "Insufficient permissions for this action"
}
```

### 404 Not Found
```json
{
  "success": false,
  "error": "Lead not found"
}
```

### 409 Conflict
```json
{
  "success": false,
  "error": "Lead with this email already exists"
}
```

### 500 Internal Server Error
```json
{
  "success": false,
  "error": "Internal server error"
}
```

---

## Rate Limiting

Coming soon: API rate limiting (100 requests per minute per IP)

---

## API Response Standards

### Success Response
```json
{
  "success": true,
  "data": { /* actual data */ },
  "message": "Operation completed"
}
```

### Error Response
```json
{
  "success": false,
  "error": "Error message",
  "details": { /* optional field-level errors */ }
}
```

## Pagination

List endpoints support pagination:
- `page`: Page number (default: 1)
- `limit`: Items per page (default: 20, max: 100)

**Response includes:**
```json
{
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "pages": 8
  }
}
```
