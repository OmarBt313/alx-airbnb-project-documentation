Backend Feature Requirement Specifications
This document outlines detailed requirements for three key backend features of the ALX Airbnb-like application, covering API endpoints, input/output specifications, validation rules, and performance criteria.

1. User Authentication & Management
This feature covers user registration, login, and basic profile management (specifically password updates).

1.1. Register Account
API Endpoint: POST /api/v1/auth/register

Description: Allows a new user to create an account.

Input (Request Body - JSON):

{
  "first_name": "string",  // VARCHAR, NOT NULL
  "last_name": "string",   // VARCHAR, NOT NULL
  "email": "string",       // VARCHAR, UNIQUE, NOT NULL
  "password": "string",    // Raw password for hashing
  "phone_number": "string",// VARCHAR, NULL
  "role": "string"         // ENUM: 'guest', 'host' (admin role created internally)
}

Validation Rules:

first_name, last_name, email, password, role are required.

email must be a valid email format and unique in the User table.

password must meet minimum strength requirements (e.g., min 8 characters, at least one uppercase, one lowercase, one number, one special character).

role must be either 'guest' or 'host'.

phone_number (if provided) must be a valid phone number format.

Output (Response Body - JSON):

Success (HTTP 201 Created):

{
  "message": "User registered successfully",
  "user_id": "uuid",
  "email": "string",
  "role": "string"
}

Error (HTTP 400 Bad Request):

{
  "error": "Invalid input data",
  "details": {
    "email": "Email already exists or invalid format",
    "password": "Password too weak"
  }
}

Error (HTTP 500 Internal Server Error):

{
  "error": "Internal server error",
  "details": "..."
}

Performance Criteria: Response time < 100 ms for 95% of requests.

1.2. Login
API Endpoint: POST /api/v1/auth/login

Description: Authenticates a user and returns an authentication token (e.g., JWT).

Input (Request Body - JSON):

{
  "email": "string",
  "password": "string"
}

Validation Rules:

email and password are required.

email must be a valid email format.

Output (Response Body - JSON):

Success (HTTP 200 OK):

{
  "message": "Login successful",
  "user_id": "uuid",
  "email": "string",
  "role": "string",
  "token": "jwt_token_string" // Authentication token
}

Error (HTTP 401 Unauthorized):

{
  "error": "Invalid credentials"
}

Error (HTTP 400 Bad Request):

{
  "error": "Missing email or password"
}

Performance Criteria: Response time < 80 ms for 95% of requests.

1.3. Update User Password
API Endpoint: PUT /api/v1/users/{user_id}/password

Description: Allows an authenticated user to change their password.

Authentication: Requires valid authentication token for the user_id being updated.

Input (Request Body - JSON):

{
  "current_password": "string",
  "new_password": "string"
}

Validation Rules:

current_password and new_password are required.

new_password must meet minimum strength requirements (different from current_password).

current_password must match the user's existing password hash.

Output (Response Body - JSON):

Success (HTTP 200 OK):

{
  "message": "Password updated successfully"
}

Error (HTTP 401 Unauthorized):

{
  "error": "Invalid current password or unauthorized access"
}

Error (HTTP 400 Bad Request):

{
  "error": "New password does not meet requirements"
}

Error (HTTP 404 Not Found):

{
  "error": "User not found"
}

Performance Criteria: Response time < 150 ms for 95% of requests.

2. Property Management
This feature allows hosts to list, view, update, and delete their properties.

2.1. List a New Property
API Endpoint: POST /api/v1/properties

Description: Allows an authenticated host to list a new property.

Authentication: Requires valid authentication token for a user with role: 'host'.

Input (Request Body - JSON):

{
  "name": "string",        // VARCHAR, NOT NULL
  "description": "string", // TEXT, NOT NULL
  "location": "string",    // VARCHAR, NOT NULL
  "pricepernight": "decimal" // DECIMAL, NOT NULL
}

Validation Rules:

name, description, location, pricepernight are required.

pricepernight must be a positive decimal number.

The authenticated user's user_id will automatically be set as host_id.

Output (Response Body - JSON):

Success (HTTP 201 Created):

{
  "message": "Property listed successfully",
  "property_id": "uuid",
  "name": "string",
  "host_id": "uuid"
}

Error (HTTP 400 Bad Request):

{
  "error": "Invalid input data",
  "details": {
    "pricepernight": "Price per night must be a positive number"
  }
}

Error (HTTP 403 Forbidden):

{
  "error": "Only hosts can list properties"
}

Performance Criteria: Response time < 200 ms for 95% of requests.

2.2. Get Property Details
API Endpoint: GET /api/v1/properties/{property_id}

Description: Retrieves detailed information about a specific property.

Authentication: Optional (publicly viewable), but authenticated users might see additional details or actions.

Input (Path Parameter): property_id (UUID)

Validation Rules:

property_id must be a valid UUID.

Output (Response Body - JSON):

Success (HTTP 200 OK):

{
  "property_id": "uuid",
  "host_id": "uuid",
  "name": "string",
  "description": "string",
  "location": "string",
  "pricepernight": "decimal",
  "created_at": "timestamp",
  "updated_at": "timestamp",
  "host_name": "string", // Derived from User table
  "average_rating": "decimal", // Derived from Review table, if any
  "total_reviews": "integer" // Derived from Review table, if any
}

Error (HTTP 404 Not Found):

{
  "error": "Property not found"
}

Error (HTTP 400 Bad Request):

{
  "error": "Invalid property ID format"
}

Performance Criteria: Response time < 100 ms for 95% of requests.

2.3. Update Property Details
API Endpoint: PUT /api/v1/properties/{property_id}

Description: Allows an authenticated host to update details of their property.

Authentication: Requires valid authentication token for the host_id of the property.

Input (Path Parameter): property_id (UUID)

Input (Request Body - JSON):

{
  "name": "string",        // Optional
  "description": "string", // Optional
  "location": "string",    // Optional
  "pricepernight": "decimal" // Optional
}

Validation Rules:

property_id must be a valid UUID.

At least one field in the request body must be provided.

pricepernight (if provided) must be a positive decimal number.

The authenticated user's user_id must match the host_id of the property.

Output (Response Body - JSON):

Success (HTTP 200 OK):

{
  "message": "Property updated successfully",
  "property_id": "uuid"
}

Error (HTTP 404 Not Found):

{
  "error": "Property not found"
}

Error (HTTP 403 Forbidden):

{
  "error": "Unauthorized: You do not own this property"
}

Error (HTTP 400 Bad Request):

{
  "error": "Invalid input data or no fields provided for update"
}

Performance Criteria: Response time < 180 ms for 95% of requests.

3. Booking System
This feature handles the creation, viewing, and management of bookings.

3.1. Create a New Booking
API Endpoint: POST /api/v1/bookings

Description: Allows an authenticated guest to create a new booking for a property.

Authentication: Requires valid authentication token for a user with role: 'guest'.

Input (Request Body - JSON):

{
  "property_id": "uuid",   // UUID, NOT NULL
  "start_date": "YYYY-MM-DD", // DATE, NOT NULL
  "end_date": "YYYY-MM-DD"   // DATE, NOT NULL
}

Validation Rules:

property_id, start_date, end_date are required.

property_id must refer to an existing property.

start_date must be today or in the future.

end_date must be after start_date.

The requested dates must not overlap with any existing confirmed bookings for that property_id.

The authenticated user's user_id will be recorded as Booking.user_id.

total_price will be calculated on the backend based on Property.pricepernight and the duration.

Initial status will be 'pending'.

Output (Response Body - JSON):

Success (HTTP 201 Created):

{
  "message": "Booking created successfully, pending confirmation",
  "booking_id": "uuid",
  "property_id": "uuid",
  "user_id": "uuid",
  "total_price": "decimal",
  "status": "pending"
}

Error (HTTP 400 Bad Request):

{
  "error": "Invalid dates or property unavailable for selected dates"
}

Error (HTTP 404 Not Found):

{
  "error": "Property not found"
}

Error (HTTP 403 Forbidden):

{
  "error": "Hosts cannot book their own properties or invalid role"
}

Performance Criteria: Response time < 300 ms for 95% of requests (includes date availability check).

3.2. Get User's Bookings (Guest/Host View)
API Endpoint:

GET /api/v1/users/{user_id}/bookings (For a guest to view their bookings)

GET /api/v1/properties/{property_id}/bookings (For a host to view bookings for their specific property)

Description: Retrieves a list of bookings associated with a specific user (as guest) or a specific property (for host).

Authentication: Requires valid authentication token.

For GET /api/v1/users/{user_id}/bookings, user_id in path must match authenticated user's ID.

For GET /api/v1/properties/{property_id}/bookings, authenticated user must be the host_id of the property.

Input (Path Parameters): user_id (UUID) or property_id (UUID)

Validation Rules:

IDs must be valid UUIDs.

Authorization check as described above.

Output (Response Body - JSON):

Success (HTTP 200 OK):

{
  "bookings": [
    {
      "booking_id": "uuid",
      "property_id": "uuid",
      "property_name": "string", // Derived from Property table
      "guest_id": "uuid",
      "guest_name": "string", // Derived from User table
      "start_date": "YYYY-MM-DD",
      "end_date": "YYYY-MM-DD",
      "total_price": "decimal",
      "status": "string",
      "created_at": "timestamp"
    }
    // ... more booking objects
  ]
}

Error (HTTP 404 Not Found):

{
  "error": "User or Property not found"
}

Error (HTTP 403 Forbidden):

{
  "error": "Unauthorized access to bookings"
}

Performance Criteria: Response time < 250 ms for 95% of requests (including joins for property/guest names).

3.3. Update Booking Status (Host Action)
API Endpoint: PUT /api/v1/bookings/{booking_id}/status

Description: Allows a host to confirm or cancel a booking request for their property.

Authentication: Requires valid authentication token for the host_id of the property associated with the booking.

Input (Path Parameter): booking_id (UUID)

Input (Request Body - JSON):

{
  "new_status": "string" // ENUM: 'confirmed', 'canceled'
}

Validation Rules:

booking_id must be a valid UUID.

new_status must be 'confirmed' or 'canceled'.

Only bookings with status: 'pending' can be moved to 'confirmed' or 'canceled'.

Only bookings with status: 'confirmed' can be moved to 'canceled'.

The authenticated user must be the host of the property related to the booking.

Output (Response Body - JSON):

Success (HTTP 200 OK):

{
  "message": "Booking status updated successfully",
  "booking_id": "uuid",
  "status": "string"
}

Error (HTTP 404 Not Found):

{
  "error": "Booking not found"
}

Error (HTTP 403 Forbidden):

{
  "error": "Unauthorized: You are not the host of this property or cannot change this booking status"
}

Error (HTTP 400 Bad Request):

{
  "error": "Invalid status transition or new_status value"
}

Performance Criteria: Response time < 150 ms for 95% of requests.