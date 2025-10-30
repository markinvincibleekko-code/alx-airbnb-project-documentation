# AirBnB Clone - Backend Requirements Specification

## Document Information
- **Project**: AirBnB Clone Backend
- **Version**: 1.0.0
- **Last Updated**: October 30, 2025
- **Author**: Backend Development Team

---

## Table of Contents
1. [User Authentication System](#1-user-authentication-system)
2. [Property Management System](#2-property-management-system)
3. [Booking System](#3-booking-system)
4. [Cross-Feature Requirements](#4-cross-feature-requirements)

---

## 1. User Authentication System

### 1.1 Overview
The User Authentication System manages user registration, login, authentication, and authorization. It ensures secure access to the platform and maintains user sessions.

### 1.2 Functional Requirements

#### FR-AUTH-001: User Registration
**Description**: Allow new users to create an account on the platform.

**API Endpoint**:
```
POST /api/v1/auth/register
```

**Input Specifications**:
```json
{
  "email": "string (required, valid email format)",
  "password": "string (required, min 8 chars, 1 uppercase, 1 lowercase, 1 number, 1 special char)",
  "first_name": "string (required, 2-50 chars, alphabetic only)",
  "last_name": "string (required, 2-50 chars, alphabetic only)",
  "phone_number": "string (optional, valid international format)",
  "role": "string (required, enum: 'guest' or 'host')"
}
```

**Output Specifications**:
```json
{
  "success": true,
  "message": "User registered successfully",
  "data": {
    "user_id": "uuid",
    "email": "string",
    "first_name": "string",
    "last_name": "string",
    "role": "string",
    "created_at": "ISO 8601 timestamp"
  },
  "token": "JWT string"
}
```

**Validation Rules**:
- Email must be unique in the system
- Email must match regex: `^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`
- Password must contain:
  - Minimum 8 characters
  - At least 1 uppercase letter
  - At least 1 lowercase letter
  - At least 1 number
  - At least 1 special character (!@#$%^&*)
- First name and last name:
  - Only alphabetic characters and spaces
  - No numbers or special characters
  - Trimmed of leading/trailing whitespace
- Phone number (if provided):
  - Must match international format: `^\+?[1-9]\d{1,14}$`
- Role must be either 'guest' or 'host'

**Error Responses**:
```json
// 400 Bad Request - Validation Error
{
  "success": false,
  "message": "Validation failed",
  "errors": [
    {
      "field": "email",
      "message": "Email is already registered"
    }
  ]
}

// 500 Internal Server Error
{
  "success": false,
  "message": "Registration failed. Please try again later."
}
```

**Business Rules**:
- Passwords must be hashed using bcrypt (cost factor: 12)
- Email verification email sent after registration
- Account inactive until email verified
- Default role is 'guest' if not specified
- User profile created automatically with basic information

**Performance Criteria**:
- Response time: < 500ms (95th percentile)
- Support 100 concurrent registrations
- Email verification sent within 2 seconds

---

#### FR-AUTH-002: User Login
**Description**: Allow registered users to authenticate and access the platform.

**API Endpoint**:
```
POST /api/v1/auth/login
```

**Input Specifications**:
```json
{
  "email": "string (required, valid email format)",
  "password": "string (required)"
}
```

**Output Specifications**:
```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "user_id": "uuid",
    "email": "string",
    "first_name": "string",
    "last_name": "string",
    "role": "string",
    "last_login": "ISO 8601 timestamp"
  },
  "token": "JWT string",
  "refresh_token": "JWT string"
}
```

**Validation Rules**:
- Email must be registered in the system
- Account must be active (email verified)
- Password must match stored hash
- Account not locked due to failed attempts

**Error Responses**:
```json
// 401 Unauthorized - Invalid Credentials
{
  "success": false,
  "message": "Invalid email or password"
}

// 403 Forbidden - Account Not Verified
{
  "success": false,
  "message": "Please verify your email before logging in",
  "data": {
    "email_verified": false
  }
}

// 423 Locked - Too Many Failed Attempts
{
  "success": false,
  "message": "Account locked due to multiple failed login attempts. Try again in 30 minutes.",
  "data": {
    "locked_until": "ISO 8601 timestamp"
  }
}
```

**Business Rules**:
- JWT token expires in 1 hour
- Refresh token expires in 7 days
- Maximum 5 failed login attempts before 30-minute lockout
- Failed attempts reset after successful login
- Last login timestamp updated on successful authentication
- Session created and stored in Redis cache

**Performance Criteria**:
- Response time: < 300ms (95th percentile)
- Support 500 concurrent logins
- Token generation: < 50ms

---

#### FR-AUTH-003: Token Refresh
**Description**: Allow users to refresh their access token using a refresh token.

**API Endpoint**:
```
POST /api/v1/auth/refresh
```

**Input Specifications**:
```json
{
  "refresh_token": "string (required, valid JWT)"
}
```

**Output Specifications**:
```json
{
  "success": true,
  "message": "Token refreshed successfully",
  "token": "JWT string (new access token)",
  "refresh_token": "JWT string (new refresh token)"
}
```

**Validation Rules**:
- Refresh token must be valid and not expired
- Refresh token must not be blacklisted
- User account must still be active

**Business Rules**:
- Old refresh token invalidated after use
- New token pair issued with fresh expiration times
- Rate limit: 10 refresh requests per hour per user

**Performance Criteria**:
- Response time: < 200ms (95th percentile)

---

#### FR-AUTH-004: Logout
**Description**: Allow users to invalidate their session and tokens.

**API Endpoint**:
```
POST /api/v1/auth/logout
```

**Headers**:
```
Authorization: Bearer {access_token}
```

**Input Specifications**:
```json
{
  "refresh_token": "string (optional)"
}
```

**Output Specifications**:
```json
{
  "success": true,
  "message": "Logged out successfully"
}
```

**Business Rules**:
- Access token added to blacklist
- Refresh token invalidated if provided
- Session removed from Redis cache
- All active sessions terminated if 'logout_all_devices' parameter set to true

**Performance Criteria**:
- Response time: < 100ms (95th percentile)

---

### 1.3 Non-Functional Requirements

#### Security Requirements
- **NFR-AUTH-SEC-001**: All passwords stored using bcrypt with cost factor 12
- **NFR-AUTH-SEC-002**: JWT tokens signed with RS256 algorithm
- **NFR-AUTH-SEC-003**: Implement rate limiting: 5 requests per minute per IP for login
- **NFR-AUTH-SEC-004**: HTTPS required for all authentication endpoints
- **NFR-AUTH-SEC-005**: Implement CSRF protection for state-changing operations
- **NFR-AUTH-SEC-006**: Log all authentication attempts (success and failure)

#### Performance Requirements
- **NFR-AUTH-PERF-001**: Support 1000 concurrent authenticated users
- **NFR-AUTH-PERF-002**: Token validation: < 10ms
- **NFR-AUTH-PERF-003**: Database queries optimized with proper indexing

#### Availability Requirements
- **NFR-AUTH-AVAIL-001**: 99.9% uptime for authentication services
- **NFR-AUTH-AVAIL-002**: Graceful degradation if email service unavailable

---

## 2. Property Management System

### 2.1 Overview
The Property Management System allows hosts to create, update, and manage property listings. It handles property details, amenities, pricing, and availability.

### 2.2 Functional Requirements

#### FR-PROP-001: Create Property Listing
**Description**: Allow hosts to create new property listings.

**API Endpoint**:
```
POST /api/v1/properties
```

**Headers**:
```
Authorization: Bearer {access_token}
Content-Type: multipart/form-data
```

**Input Specifications**:
```json
{
  "title": "string (required, 10-100 chars)",
  "description": "string (required, 50-2000 chars)",
  "location": {
    "address": "string (required)",
    "city": "string (required)",
    "state": "string (required)",
    "country": "string (required)",
    "postal_code": "string (required)",
    "latitude": "decimal (required, -90 to 90)",
    "longitude": "decimal (required, -180 to 180)"
  },
  "property_type": "string (required, enum: apartment, house, villa, condo, cabin, etc.)",
  "pricing": {
    "base_price": "decimal (required, min 1, max 100000)",
    "currency": "string (required, ISO 4217 code, default: USD)",
    "cleaning_fee": "decimal (optional, min 0)",
    "service_fee_percentage": "decimal (calculated, default 15%)"
  },
  "capacity": {
    "guests": "integer (required, min 1, max 16)",
    "bedrooms": "integer (required, min 0)",
    "beds": "integer (required, min 1)",
    "bathrooms": "decimal (required, min 0.5, step 0.5)"
  },
  "amenities": [
    "string (array of amenity IDs)"
  ],
  "house_rules": "string (optional, max 1000 chars)",
  "cancellation_policy": "string (required, enum: flexible, moderate, strict)",
  "images": "file[] (required, min 3, max 20, jpg/png, max 5MB each)"
}
```

**Output Specifications**:
```json
{
  "success": true,
  "message": "Property created successfully",
  "data": {
    "property_id": "uuid",
    "host_id": "uuid",
    "title": "string",
    "description": "string",
    "location": {
      "address": "string",
      "city": "string",
      "state": "string",
      "country": "string",
      "postal_code": "string",
      "coordinates": {
        "latitude": "decimal",
        "longitude": "decimal"
      }
    },
    "property_type": "string",
    "pricing": {
      "base_price": "decimal",
      "currency": "string",
      "cleaning_fee": "decimal",
      "service_fee_percentage": "decimal"
    },
    "capacity": {
      "guests": "integer",
      "bedrooms": "integer",
      "beds": "integer",
      "bathrooms": "decimal"
    },
    "amenities": [
      {
        "amenity_id": "uuid",
        "name": "string",
        "icon": "string"
      }
    ],
    "images": [
      {
        "image_id": "uuid",
        "url": "string",
        "is_primary": "boolean",
        "order": "integer"
      }
    ],
    "house_rules": "string",
    "cancellation_policy": "string",
    "status": "string (pending_review)",
    "created_at": "ISO 8601 timestamp"
  }
}
```

**Validation Rules**:
- Title:
  - Length: 10-100 characters
  - No special characters except hyphens and commas
- Description:
  - Length: 50-2000 characters
  - Basic HTML sanitization
- Location:
  - Address geocoded to verify coordinates
  - Country must be from supported countries list
  - Postal code format validated based on country
- Pricing:
  - Base price must be positive
  - Currency must be supported (USD, EUR, GBP, etc.)
  - Cleaning fee cannot exceed base price
- Capacity:
  - Guests must be realistic (1-16)
  - Bedrooms can be 0 (studio apartment)
  - Beds must be at least 1
  - Bathrooms in 0.5 increments
- Images:
  - Minimum 3 images required
  - Maximum 20 images
  - Formats: JPEG, PNG only
  - Max size: 5MB per image
  - First image set as primary by default
- Amenities:
  - Must reference valid amenity IDs from amenities table
  - Maximum 30 amenities per property
- Cancellation policy must be one of: flexible, moderate, strict

**Error Responses**:
```json
// 400 Bad Request - Validation Error
{
  "success": false,
  "message": "Validation failed",
  "errors": [
    {
      "field": "images",
      "message": "At least 3 images are required"
    }
  ]
}

// 401 Unauthorized
{
  "success": false,
  "message": "Authentication required"
}

// 403 Forbidden - Not a Host
{
  "success": false,
  "message": "Only hosts can create property listings"
}

// 413 Payload Too Large
{
  "success": false,
  "message": "Image file size exceeds 5MB limit"
}
```

**Business Rules**:
- Only users with 'host' role can create properties
- Property status set to 'pending_review' initially
- Admin approval required before property is listed
- Images uploaded to cloud storage (AWS S3/Cloudinary)
- Geocoding performed on address to get coordinates
- Service fee calculated automatically (15% of base price)
- Property automatically inactive until approved

**Performance Criteria**:
- Response time: < 2 seconds (including image upload)
- Image upload: < 500ms per image (parallel processing)
- Support 50 concurrent property creations
- Database transaction for data consistency

---

#### FR-PROP-002: Update Property Listing
**Description**: Allow hosts to update their existing property listings.

**API Endpoint**:
```
PUT /api/v1/properties/{property_id}
PATCH /api/v1/properties/{property_id}
```

**Headers**:
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

**Input Specifications**:
- Same as FR-PROP-001, but all fields optional
- Partial updates supported with PATCH

**Output Specifications**:
- Same as FR-PROP-001 with updated data

**Validation Rules**:
- All validation rules from FR-PROP-001 apply
- Property must exist and belong to authenticated host
- Cannot update property_id or host_id
- If images updated, minimum 3 images must be maintained

**Business Rules**:
- Property goes to 'pending_review' status if major changes made
- Major changes: pricing change > 20%, location change, capacity change
- Minor changes: description, amenities, images
- Version history maintained for audit trail

**Performance Criteria**:
- Response time: < 1 second (without image changes)
- Response time: < 2 seconds (with image changes)

---

#### FR-PROP-003: Delete Property Listing
**Description**: Allow hosts to delete their property listings.

**API Endpoint**:
```
DELETE /api/v1/properties/{property_id}
```

**Headers**:
```
Authorization: Bearer {access_token}
```

**Output Specifications**:
```json
{
  "success": true,
  "message": "Property deleted successfully",
  "data": {
    "property_id": "uuid",
    "deleted_at": "ISO 8601 timestamp"
  }
}
```

**Validation Rules**:
- Property must exist and belong to authenticated host
- Cannot delete if active bookings exist

**Error Responses**:
```json
// 400 Bad Request - Active Bookings
{
  "success": false,
  "message": "Cannot delete property with active bookings",
  "data": {
    "active_bookings_count": 3
  }
}

// 404 Not Found
{
  "success": false,
  "message": "Property not found"
}
```

**Business Rules**:
- Soft delete: property marked as deleted, not physically removed
- Property data retained for 90 days for recovery
- All future bookings automatically canceled
- Past bookings and reviews preserved
- Images moved to archive storage

**Performance Criteria**:
- Response time: < 300ms

---

#### FR-PROP-004: Get Property Details
**Description**: Retrieve detailed information about a specific property.

**API Endpoint**:
```
GET /api/v1/properties/{property_id}
```

**Query Parameters**:
```
?include=host,reviews,amenities (optional)
```

**Output Specifications**:
```json
{
  "success": true,
  "data": {
    "property_id": "uuid",
    "host": {
      "user_id": "uuid",
      "first_name": "string",
      "profile_image": "string",
      "joined_date": "ISO 8601 timestamp",
      "is_superhost": "boolean",
      "response_rate": "decimal",
      "response_time": "string"
    },
    "title": "string",
    "description": "string",
    "location": { },
    "property_type": "string",
    "pricing": { },
    "capacity": { },
    "amenities": [ ],
    "images": [ ],
    "house_rules": "string",
    "cancellation_policy": "string",
    "rating": {
      "average": "decimal (0-5)",
      "count": "integer",
      "breakdown": {
        "cleanliness": "decimal",
        "accuracy": "decimal",
        "communication": "decimal",
        "location": "decimal",
        "check_in": "decimal",
        "value": "decimal"
      }
    },
    "reviews": [
      {
        "review_id": "uuid",
        "guest_name": "string",
        "rating": "integer",
        "comment": "string",
        "created_at": "ISO 8601 timestamp"
      }
    ],
    "availability": {
      "min_nights": "integer",
      "max_nights": "integer",
      "booking_notice": "integer (hours)",
      "blocked_dates": ["date array"]
    },
    "created_at": "ISO 8601 timestamp",
    "updated_at": "ISO 8601 timestamp"
  }
}
```

**Business Rules**:
- Only active/published properties visible to guests
- Hosts can view their own properties regardless of status
- View count incremented on each access
- Recently viewed properties tracked for recommendations

**Performance Criteria**:
- Response time: < 200ms (with caching)
- Cache TTL: 5 minutes
- CDN delivery for images

---

#### FR-PROP-005: Search Properties
**Description**: Search and filter properties based on various criteria.

**API Endpoint**:
```
GET /api/v1/properties/search
```

**Query Parameters**:
```
?location=string (city, state, or country)
&check_in=YYYY-MM-DD (required)
&check_out=YYYY-MM-DD (required)
&guests=integer (required, default: 1)
&min_price=decimal (optional)
&max_price=decimal (optional)
&property_type=string (optional, comma-separated)
&amenities=string (optional, comma-separated amenity IDs)
&min_bedrooms=integer (optional)
&min_bathrooms=decimal (optional)
&min_rating=decimal (optional, 0-5)
&instant_book=boolean (optional)
&superhost=boolean (optional)
&page=integer (default: 1)
&limit=integer (default: 20, max: 100)
&sort_by=string (price_asc, price_desc, rating, newest)
```

**Output Specifications**:
```json
{
  "success": true,
  "data": {
    "properties": [
      {
        "property_id": "uuid",
        "title": "string",
        "location": {
          "city": "string",
          "state": "string",
          "country": "string"
        },
        "pricing": {
          "base_price": "decimal",
          "total_price": "decimal (for selected dates)",
          "currency": "string"
        },
        "capacity": {
          "guests": "integer",
          "bedrooms": "integer",
          "bathrooms": "decimal"
        },
        "rating": {
          "average": "decimal",
          "count": "integer"
        },
        "primary_image": "string (URL)",
        "is_superhost": "boolean",
        "instant_book": "boolean"
      }
    ],
    "pagination": {
      "current_page": "integer",
      "total_pages": "integer",
      "total_results": "integer",
      "results_per_page": "integer"
    },
    "filters_applied": {
      "location": "string",
      "date_range": {
        "check_in": "date",
        "check_out": "date"
      },
      "guests": "integer"
    }
  }
}
```

**Validation Rules**:
- Check-in date must be today or future date
- Check-out date must be after check-in date
- Maximum stay: 365 days
- Guests must be positive integer
- Price range: min_price < max_price
- Rating must be 0-5

**Business Rules**:
- Only available properties returned
- Properties checked against booking calendar
- Results ranked by relevance algorithm
- Price includes all nights + fees
- Search results cached for common queries

**Performance Criteria**:
- Response time: < 500ms (with database optimization)
- Elasticsearch for full-text search
- Database indexes on: location, price, rating, availability
- Support 200 concurrent searches
- Cache popular search queries for 10 minutes

---

### 2.3 Non-Functional Requirements

#### Security Requirements
- **NFR-PROP-SEC-001**: Only property owners can update/delete their listings
- **NFR-PROP-SEC-002**: Image uploads scanned for malware
- **NFR-PROP-SEC-003**: Input sanitization to prevent XSS attacks
- **NFR-PROP-SEC-004**: Rate limiting: 100 requests per hour per user

#### Performance Requirements
- **NFR-PROP-PERF-001**: Search results cached in Redis
- **NFR-PROP-PERF-002**: Images served via CDN
- **NFR-PROP-PERF-003**: Database query optimization with proper indexes
- **NFR-PROP-PERF-004**: Lazy loading for property images

#### Data Requirements
- **NFR-PROP-DATA-001**: Property data backup daily
- **NFR-PROP-DATA-002**: Image thumbnails generated automatically
- **NFR-PROP-DATA-003**: Soft delete with 90-day retention

---

## 3. Booking System

### 3.1 Overview
The Booking System manages the entire booking lifecycle from property reservation to payment processing and confirmation.

### 3.2 Functional Requirements

#### FR-BOOK-001: Create Booking
**Description**: Allow guests to create a booking for an available property.

**API Endpoint**:
```
POST /api/v1/bookings
```

**Headers**:
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

**Input Specifications**:
```json
{
  "property_id": "uuid (required)",
  "check_in": "date (required, YYYY-MM-DD)",
  "check_out": "date (required, YYYY-MM-DD)",
  "guests": {
    "adults": "integer (required, min 1)",
    "children": "integer (optional, default 0)",
    "infants": "integer (optional, default 0)"
  },
  "special_requests": "string (optional, max 500 chars)",
  "payment_method": "string (required, enum: credit_card, stripe, paypal)"
}
```

**Output Specifications**:
```json
{
  "success": true,
  "message": "Booking created successfully",
  "data": {
    "booking_id": "uuid",
    "property_id": "uuid",
    "guest_id": "uuid",
    "host_id": "uuid",
    "booking_details": {
      "check_in": "date",
      "check_out": "date",
      "nights": "integer",
      "guests": {
        "adults": "integer",
        "children": "integer",
        "infants": "integer",
        "total": "integer"
      }
    },
    "pricing": {
      "base_price": "decimal",
      "nights_cost": "decimal",
      "cleaning_fee": "decimal",
      "service_fee": "decimal",
      "taxes": "decimal",
      "total_amount": "decimal",
      "currency": "string"
    },
    "status": "string (pending_payment)",
    "payment": {
      "payment_id": "uuid",
      "status": "pending",
      "payment_method": "string"
    },
    "cancellation_policy": "string",
    "special_requests": "string",
    "created_at": "ISO 8601 timestamp",
    "expires_at": "ISO 8601 timestamp (15 minutes from creation)"
  }
}
```

**Validation Rules**:
- Property must exist and be active
- Check-in date must be today or future
- Check-out date must be after check-in
- Total guests cannot exceed property capacity
- Adults + children + infants <= property.capacity.guests
- Infants (under 2 years) don't count toward capacity
- Property must be available for selected dates
- Check-in/check-out must respect property's min/max night requirements
- Booking notice period must be respected

**Error Responses**:
```json
// 400 Bad Request - Validation Error
{
  "success": false,
  "message": "Booking validation failed",
  "errors": [
    {
      "field": "check_in",
      "message": "Property requires 24 hours booking notice"
    }
  ]
}

// 409 Conflict - Dates Not Available
{
  "success": false,
  "message": "Property not available for selected dates",
  "data": {
    "available_dates": [
      {
        "check_in": "date",
        "check_out": "date"
      }
    ]
  }
}

// 402 Payment Required
{
  "success": false,
  "message": "Payment processing failed",
  "data": {
    "booking_id": "uuid",
    "payment_error": "string"
  }
}
```

**Business Rules**:
- Booking status initially set to 'pending_payment'
- Property calendar blocked for 15 minutes during booking process
- If payment not completed in 15 minutes, booking expires
- Guest cannot book their own property
- Simultaneous booking requests handled with row-level locking
- Price calculated dynamically based on:
  - Nightly rate × number of nights
  - Cleaning fee (one-time)
  - Service fee (15% of subtotal)
  - Taxes (based on location)
- Confirmation email sent to guest and notification to host
- Calendar updated to block booked dates

**Performance Criteria**:
- Response time: < 1 second
- Database transaction for atomicity
- Redis lock for preventing double bookings
- Support 100 concurrent booking requests

---

#### FR-BOOK-002: Process Payment
**Description**: Process payment for a pending booking.

**API Endpoint**:
```
POST /api/v1/bookings/{booking_id}/payment
```

**Headers**:
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

**Input Specifications**:
```json
{
  "payment_method": "string (required)",
  "payment_details": {
    // For credit card
    "card_number": "string (required for card, 16 digits)",
    "expiry_month": "integer (required for card, 1-12)",
    "expiry_year": "integer (required for card, YYYY)",
    "cvv": "string (required for card, 3-4 digits)",
    "cardholder_name": "string (required for card)",
    
    // For Stripe
    "stripe_token": "string (required for stripe)",
    
    // For PayPal
    "paypal_order_id": "string (required for paypal)"
  },
  "billing_address": {
    "street": "string (required)",
    "city": "string (required)",
    "state": "string (required)",
    "postal_code": "string (required)",
    "country": "string (required)"
  }
}
```

**Output Specifications**:
```json
{
  "success": true,
  "message": "Payment processed successfully",
  "data": {
    "payment_id": "uuid",
    "booking_id": "uuid",
    "amount": "decimal",
    "currency": "string",
    "payment_method": "string",
    "transaction_id": "string",
    "status": "completed",
    "processed_at": "ISO 8601 timestamp",
    "receipt_url": "string"
  }
}
```

**Validation Rules**:
- Booking must exist and be in 'pending_payment' status
- Booking must not be expired (within 15 minutes)
- Payment amount must match booking total
- Credit card validation:
  - Card number: valid Luhn algorithm
  - Expiry date: not expired
  - CVV: 3-4 digits
- Billing address required for all payment methods

**Error Responses**:
```json
// 402 Payment Required - Payment Failed
{
  "success": false,
  "message": "Payment declined",
  "data": {
    "error_code": "insufficient_funds",
    "error_message": "Your card has insufficient funds"
  }
}

// 410 Gone - Booking Expired
{
  "success": false,
  "message": "Booking has expired. Please create a new booking."
}
```

**Business Rules**:
- Payment processed through payment gateway (Stripe/PayPal)
- Successful payment updates booking status to 'confirmed'
- Failed payment updates booking status to 'payment_failed'
- 3 payment retry attempts allowed
- Payment transactions logged for audit
- Payout to host scheduled (release after check-in + 24 hours)
- Payment confirmation email sent immediately
- Refunds follow cancellation policy

**Performance Criteria**:
- Response time: < 3 seconds (including gateway processing)
- Payment gateway timeout: 30 seconds
- Automatic retry for network failures

---

#### FR-BOOK-003: Get Booking Details
**Description**: Retrieve detailed information about a specific booking.

**API Endpoint**:
```
GET /api/v1/bookings/{booking_id}
```

**Headers**:
```
Authorization: Bearer {access_token}
```

**Output Specifications**:
```json
{
  "success": true,
  "data": {
    "