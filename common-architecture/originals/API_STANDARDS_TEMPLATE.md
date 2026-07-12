# API Standards

**Project:** [PROJECT_NAME]  
**API Version:** [v1/v2]  
**Last Updated:** [DATE]

---

## Overview

This document defines REST API standards for [PROJECT_NAME]. All APIs must follow these standards unless documented exceptions exist.

---

## URL Structure

### Versioning Strategy

**Approach:** [URL path / Header / Query param - CHOOSE ONE]

```
URL Path (Recommended):
GET /api/v1/users
GET /api/v2/users

Header:
GET /api/users
Header: Accept: application/vnd.myapp.v2+json

Query Parameter:
GET /api/users?version=2
```

### Base URL

```
Development:   http://localhost:[PORT]/api
Staging:       https://staging-api.example.com/api
Production:    https://api.example.com/api
```

### Naming Conventions

**Resource URLs:**
- Use plural nouns: `/users` not `/user`
- Use hyphens for multi-word resources: `/user-profiles` not `/userProfiles`
- Use path parameters for relationships: `/users/{id}/posts` not `/posts?userId=123`

**Examples:**
```
✅ GET /api/v1/users                    # List all users
✅ GET /api/v1/users/{id}               # Get specific user
✅ GET /api/v1/users/{id}/posts         # Get user's posts
✅ POST /api/v1/users                   # Create user
✅ PUT /api/v1/users/{id}               # Full update
✅ PATCH /api/v1/users/{id}             # Partial update
✅ DELETE /api/v1/users/{id}            # Delete user

❌ GET /api/user                        # Don't use singular
❌ GET /api/getUser                     # Don't repeat in method
❌ GET /api/users?action=list           # Avoid action params
```

---

## HTTP Methods

| Method | Purpose | Has Body | Idempotent | Cacheable |
|--------|---------|----------|-----------|-----------|
| GET | Retrieve resource | No | ✅ Yes | ✅ Yes |
| POST | Create resource | ✅ Yes | ❌ No | ❌ No |
| PUT | Full replacement | ✅ Yes | ✅ Yes | ❌ No |
| PATCH | Partial update | ✅ Yes | ❌ No | ❌ No |
| DELETE | Remove resource | No | ✅ Yes | ❌ No |
| HEAD | Like GET, no body | No | ✅ Yes | ✅ Yes |
| OPTIONS | Describe available methods | No | ✅ Yes | ✅ Yes |

### Method Rules

**GET:**
- Safe (no side effects)
- Idempotent (same result on repeat)
- No request body
- Cacheable for 300 seconds (default)

**POST:**
- Creates new resource
- Returns 201 Created with Location header
- Not idempotent (multiple POSTs = multiple resources)

**PUT:**
- Replaces entire resource
- Requires all fields
- Idempotent (repeat = same result)
- Returns 200 OK or 204 No Content

**PATCH:**
- Partial update
- Requires only changed fields
- May not be idempotent
- Returns 200 OK with updated resource

**DELETE:**
- Removes resource
- Idempotent (delete twice = same result)
- Returns 204 No Content
- May soft-delete if data audit required

---

## Request Format

### Headers

**Required headers:**
```
Content-Type: application/json
Accept: application/json
Authorization: Bearer [TOKEN]           # If applicable
```

**Optional headers:**
```
X-Request-ID: [UUID]                    # For tracing
X-Correlation-ID: [UUID]                # For related requests
User-Agent: [Client/Version]            # Client identification
```

### Request Body

**Content-Type:** Always `application/json`

**Structure:**
```json
{
  "field_name": "value",
  "nested_object": {
    "property": "value"
  },
  "array_field": [1, 2, 3]
}
```

**Rules:**
- Use `snake_case` for field names (not camelCase)
- Include only necessary fields
- Use null for optional unset fields
- Validate all inputs server-side

### Query Parameters

**Pagination:**
```
GET /api/v1/users?page=1&limit=20
GET /api/v1/users?offset=0&limit=20
```

**Filtering:**
```
GET /api/v1/users?status=active&role=admin
GET /api/v1/users?name_like=john
GET /api/v1/posts?created_after=2026-01-01&created_before=2026-12-31
```

**Sorting:**
```
GET /api/v1/users?sort=name              # Ascending
GET /api/v1/users?sort=-created_at       # Descending (- prefix)
GET /api/v1/users?sort=name,created_at   # Multiple fields
```

**Field selection:**
```
GET /api/v1/users/{id}?fields=id,name,email   # Sparse fields
```

**Standards:**
- Limit query params to essential filters
- Use standard names: `page`, `limit`, `sort`, `fields`
- Document default values
- Validate and sanitize all params

---

## Response Format

### Success Response Structure

**200 OK (GET/PUT/PATCH):**
```json
{
  "status": "success",
  "code": 200,
  "message": "Operation completed",
  "data": {
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com"
  },
  "metadata": {
    "timestamp": "2026-07-12T12:00:00Z",
    "version": "1.0",
    "correlation_id": "req-12345"
  }
}
```

**201 Created (POST):**
```json
{
  "status": "success",
  "code": 201,
  "message": "Resource created",
  "data": {
    "id": 456,
    "name": "New User"
  }
}
```

**204 No Content (DELETE/empty response):**
```
HTTP/1.1 204 No Content
```

**List Response with Pagination:**
```json
{
  "status": "success",
  "code": 200,
  "data": [
    { "id": 1, "name": "User 1" },
    { "id": 2, "name": "User 2" }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 450,
    "total_pages": 23,
    "has_next": true,
    "has_previous": false
  }
}
```

### Error Response Structure

**Generic error format:**
```json
{
  "status": "error",
  "code": [HTTP_STATUS],
  "message": "[User-friendly message]",
  "error": {
    "type": "[ERROR_CODE]",
    "details": "[Additional context]"
  },
  "metadata": {
    "timestamp": "2026-07-12T12:00:00Z",
    "correlation_id": "req-12345",
    "path": "/api/v1/users/123"
  }
}
```

**Validation error:**
```json
{
  "status": "error",
  "code": 400,
  "message": "Validation failed",
  "errors": [
    {
      "field": "email",
      "message": "Invalid email format",
      "code": "INVALID_EMAIL"
    },
    {
      "field": "password",
      "message": "Must be at least 8 characters",
      "code": "PASSWORD_TOO_SHORT"
    }
  ]
}
```

### HTTP Status Codes

| Code | Meaning | When to Use |
|------|---------|-----------|
| **2xx - Success** |
| 200 | OK | Standard successful request |
| 201 | Created | New resource created (POST) |
| 204 | No Content | Successful delete or empty response |
| **3xx - Redirection** |
| 301 | Moved Permanently | Resource moved (rare) |
| 302 | Found | Temporary redirect |
| 304 | Not Modified | Client cache is current |
| **4xx - Client Error** |
| 400 | Bad Request | Invalid input/format |
| 401 | Unauthorized | Authentication required |
| 403 | Forbidden | Authenticated but not allowed |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Version conflict, duplicate key |
| 429 | Too Many Requests | Rate limited |
| **5xx - Server Error** |
| 500 | Internal Server Error | Unexpected error |
| 502 | Bad Gateway | Upstream service error |
| 503 | Service Unavailable | Maintenance or overload |
| 504 | Gateway Timeout | Timeout waiting for response |

**Use cases:**
- ✅ 400 for validation errors
- ✅ 401 for missing authentication
- ✅ 403 for insufficient permissions
- ✅ 404 for missing resources
- ✅ 409 for conflicts (duplicate, version mismatch)
- ✅ 429 for rate limiting
- ❌ Don't return 500 for known errors

---

## Authentication & Authorization

### Authentication Headers

```
Authorization: Bearer [JWT_TOKEN]
```

**Token format:**
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIn0.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

**Included in all requests:**
```
curl -H "Authorization: Bearer [TOKEN]" https://api.example.com/api/v1/users
```

### Rate Limiting Headers

**Response headers:**
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 995
X-RateLimit-Reset: 1626200400
```

**When limit exceeded (429):**
```json
{
  "status": "error",
  "code": 429,
  "message": "Rate limit exceeded",
  "retry_after": 60
}
```

---

## Data Types & Formats

### Timestamps

**Format:** ISO 8601 with timezone

```json
{
  "created_at": "2026-07-12T12:00:00Z",
  "updated_at": "2026-07-12T13:30:45.123Z"
}
```

**NOT:**
```
"created_at": "2026-07-12"              // Missing time
"created_at": "7/12/2026 12:00 PM"      // Wrong format
"created_at": 1626100800000             // Milliseconds timestamp
```

### Booleans

```json
{
  "is_active": true,
  "is_deleted": false,
  "verified": true
}
```

**NOT:** Strings like `"true"`, `"yes"`, `1`

### Null Values

```json
{
  "name": "John",
  "middle_name": null,
  "phone": null
}
```

**Rules:**
- Use `null` for unset optional fields
- Omit field entirely if API contract doesn't require it
- Never use empty string `""` to represent null

### Enums

```json
{
  "status": "active",
  "role": "admin"
}
```

**Document all valid values:**
```
status: "active" | "inactive" | "suspended"
role: "admin" | "user" | "guest"
```

### UUIDs

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000"
}
```

**NOT:** Sequential integers or random strings

### Dates (date-only)

```json
{
  "birth_date": "1990-05-15",
  "start_date": "2026-01-01"
}
```

**Format:** YYYY-MM-DD (no time component)

---

## Pagination Standards

**Query parameters:**
```
page=1&limit=20        # Offset-based
offset=0&limit=20      # Offset-based (alternative)
cursor=[TOKEN]&limit=20 # Cursor-based (for large datasets)
```

**Response structure:**
```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 500,
    "total_pages": 25,
    "has_next": true,
    "has_previous": false
  }
}
```

**Limits:**
- Minimum page size: 1
- Maximum page size: 100 (prevent abuse)
- Default page size: 20
- Large datasets: Use cursor-based pagination

---

## Versioning Strategy

### Deprecation Policy

**Timeline:**
1. Announce deprecation (at least 6 months notice)
2. Mark endpoint as deprecated in docs and headers
3. Support old version while new version exists
4. Remove after deprecation period

**Header warning:**
```
Deprecation: true
Sunset: Sun, 31 Dec 2026 23:59:59 GMT
Link: </api/v2/users>; rel="successor-version"
```

### Version Migration

**v1 → v2 migration process:**
```
1. Client updates to consume v2
2. Client tests thoroughly
3. Client updates production
4. API team removes v1 after all clients migrated
```

---

## Error Codes

**Standard error codes:**
```
VALIDATION_ERROR        # Invalid input
AUTHENTICATION_ERROR    # Missing/invalid token
AUTHORIZATION_ERROR     # Insufficient permissions
NOT_FOUND              # Resource doesn't exist
CONFLICT               # Version conflict or duplicate
RATE_LIMITED           # Too many requests
SERVICE_UNAVAILABLE    # Maintenance or overload
INTERNAL_ERROR         # Unexpected server error
```

**Example:**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Email is required",
    "details": {
      "field": "email"
    }
  }
}
```

---

## Documentation Requirements

**Every endpoint must document:**
- Purpose (one sentence)
- Request format with examples
- Response format with examples
- Possible error codes
- Rate limiting (if different from default)
- Required permissions
- Example cURL command

**Example:**
```markdown
### Get User by ID

Retrieves a specific user by their ID.

**Endpoint:**
```
GET /api/v1/users/{id}
```

**Request:**
```
curl -H "Authorization: Bearer [TOKEN]" \
  https://api.example.com/api/v1/users/123
```

**Response (200):**
```json
{
  "status": "success",
  "data": {
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com"
  }
}
```

**Error Responses:**
- 401: Unauthorized
- 403: Forbidden (no permission)
- 404: User not found
- 429: Rate limited
```

---

## Testing Checklist

Before shipping an API:

- [ ] All requests validate input
- [ ] All responses match documented format
- [ ] Error responses include proper codes
- [ ] Rate limiting is configured
- [ ] Authentication is required
- [ ] Authorization is checked
- [ ] Sensitive data is not logged
- [ ] CORS headers are correct
- [ ] Load testing passes
- [ ] Security review completed
- [ ] Documentation is complete
- [ ] Example cURL commands work

---

## Quick Reference

**Always:**
- ✅ Use HTTPS in production
- ✅ Validate all inputs
- ✅ Return appropriate status codes
- ✅ Include correlation IDs
- ✅ Document error cases
- ✅ Support pagination for lists
- ✅ Use consistent naming
- ✅ Version your APIs

**Never:**
- ❌ Return sensitive data in error messages
- ❌ Log passwords or tokens
- ❌ Trust client-side validation
- ❌ Use GET for state changes
- ❌ Forget rate limiting
- ❌ Return HTML from REST API
- ❌ Mix REST and RPC styles
