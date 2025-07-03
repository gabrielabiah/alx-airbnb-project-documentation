# Airbnb Clone Backend Requirement Specifications

This document details the technical and functional requirements for three key backend features: User Authentication, Property Management, and Booking System. Each section includes API endpoints, input/output specifications, validation rules, and performance criteria.

---

## 1. User Authentication

### Description

Handles user registration, login, and authentication (JWT-based), supporting both guests and hosts.

### API Endpoints

#### `POST /api/v1/auth/register`

- **Description:** Register a new user (guest or host).
- **Input (JSON):**
  ```json
  {
    "name": "string",
    "email": "string",
    "password": "string",
    "role": "guest|host"
  }
  ```
- **Validation Rules:**
  - `name`: Required, min 2, max 50 characters.
  - `email`: Required, valid email format, unique.
  - `password`: Required, min 8 characters, must include a letter and a number.
  - `role`: Required, must be either "guest" or "host".
- **Output:**
  - `201 Created` with user object (without password), JWT token.
  - Error codes: `400` (validation), `409` (email exists).

#### `POST /api/v1/auth/login`

- **Description:** Login and receive JWT token.
- **Input (JSON):**
  ```json
  {
    "email": "string",
    "password": "string"
  }
  ```
- **Validation Rules:**
  - `email`: Required, valid email.
  - `password`: Required.
- **Output:**
  - `200 OK` with user data (without password), JWT token.
  - Error codes: `400` (validation), `401` (invalid credentials).

#### `GET /api/v1/auth/profile`

- **Description:** Returns authenticated user profile.
- **Authentication:** JWT Bearer Token required.
- **Output:**
  - `200 OK` with user profile data.

### Performance Criteria

- Registration and login must complete in < 500ms under normal load.
- Passwords stored as secure hashes (e.g., bcrypt).
- Rate limiting on authentication endpoints: max 5 attempts/minute per IP.

---

## 2. Property Management

### Description

Enables hosts to add, edit, delete, and retrieve property listings. Guests can search and view listings.

### API Endpoints

#### `POST /api/v1/properties`

- **Description:** Add a new property listing (host only).
- **Input (JSON):**
  ```json
  {
    "title": "string",
    "description": "string",
    "location": "string",
    "price": float,
    "amenities": ["string"],
    "images": ["url"],
    "availability": [
      {"start_date": "YYYY-MM-DD", "end_date": "YYYY-MM-DD"}
    ]
  }
  ```
- **Validation Rules:**
  - All fields required except `images`.
  - `price`: Must be > 0.
  - `availability`: Dates must be valid, non-overlapping.
- **Authentication:** JWT, must be host.
- **Output:**
  - `201 Created` with property object.
  - Error codes: `400` (validation), `401` (unauthenticated), `403` (not host).

#### `GET /api/v1/properties`

- **Description:** Search or list properties (supports filters, pagination).
- **Query Parameters:**
  - `location`, `min_price`, `max_price`, `guests`, `amenities`, `page`, `limit`
- **Output:**
  - `200 OK` with array of property objects and pagination metadata.

#### `PUT /api/v1/properties/{id}`

- **Description:** Edit a property (host & owner only).
- **Input:** Same as POST.
- **Output:**
  - `200 OK` with updated property.
  - Error codes: `403` (not owner).

#### `DELETE /api/v1/properties/{id}`

- **Description:** Delete a property (host & owner only).
- **Output:**
  - `204 No Content`
  - Error codes: `403` (not owner), `404` (not found).

### Performance Criteria

- Property search must support pagination and respond in < 1s for up to 10,000 listings.
- Image uploads handled asynchronously; URLs stored in property records.
- Consistent response times under high traffic via query optimization and caching.

---

## 3. Booking System

### Description

Allows guests to book properties, prevents double bookings, and manages booking statuses.

### API Endpoints

#### `POST /api/v1/bookings`

- **Description:** Create a new booking for a property.
- **Input (JSON):**
  ```json
  {
    "property_id": "uuid",
    "start_date": "YYYY-MM-DD",
    "end_date": "YYYY-MM-DD",
    "guests": integer
  }
  ```
- **Validation Rules:**
  - All fields required.
  - `start_date` must be before `end_date`.
  - Dates must not overlap existing bookings.
  - `guests`: Must not exceed property capacity.
- **Authentication:** JWT, must be guest.
- **Output:**
  - `201 Created` with booking object and status `pending`.
  - Error codes: `400` (validation), `409` (dates unavailable).

#### `GET /api/v1/bookings/{id}`

- **Description:** Get booking details (owner or guest only).
- **Output:**
  - `200 OK` with booking details.
  - Error codes: `403` (not allowed), `404` (not found).

#### `PATCH /api/v1/bookings/{id}/cancel`

- **Description:** Cancel a booking (by guest or host per policy).
- **Output:**
  - `200 OK` with updated booking status `cancelled`.
  - Error codes: `403` (not allowed), `404` (not found).

### Performance Criteria

- Date/time conflicts checked at booking creation.
- All read/write operations on bookings must complete < 750ms under normal load.
- Support for up to 100,000 concurrent bookings with efficient queries and indexing.

---

## General Notes

- All endpoints to return standard HTTP codes and error messages.
- API must be RESTful, with clear versioning (`/api/v1/`).
- All endpoints protected by authentication and role-based access where appropriate.
- Validation errors must be descriptive.
- Rate limiting and logging in place for critical operations.
