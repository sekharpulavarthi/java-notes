# REST API & Web Services

## 1. What is a Web Service?

A **web service** is a software service that allows two applications to communicate with each other over a network, commonly using HTTP.

For example:

```text
React Application
       ↓ HTTP
Java Backend
       ↓
Database
```

The React application and Java backend communicate through APIs.

---

# 2. What is an API?

**API (Application Programming Interface)** is a defined way for one software application to communicate with another.

For example:

```text
GET /users/10
```

The client is asking the backend:

> Give me the user whose ID is 10.

The backend processes the request and returns a response.

---

# 3. What is a REST API?

**REST (Representational State Transfer)** is an architectural style for designing network APIs.

A REST API generally uses:

- HTTP
- URLs to represent resources
- HTTP methods to perform operations
- HTTP status codes
- JSON for data exchange

Example:

```text
GET    /users
GET    /users/10
POST   /users
PUT    /users/10
DELETE /users/10
```

---

# 4. What is a Resource?

In REST, an important concept is the **resource**.

A resource represents something in the application.

Examples:

```text
/users
/products
/orders
/transactions
/categories
```

For example:

```text
/users/10
```

represents the user with ID `10`.

REST APIs generally use **nouns**, rather than actions, in URLs.

Prefer:

```text
GET /users/10
```

rather than:

```text
GET /getUser/10
```

The HTTP method already describes the operation.

---

# 5. HTTP Methods

The most important HTTP methods are:

| Method | Common purpose                  |
| ------ | ------------------------------- |
| GET    | Retrieve data                   |
| POST   | Create a resource / submit data |
| PUT    | Replace/update a resource       |
| PATCH  | Partially update a resource     |
| DELETE | Delete a resource               |

Example for a `users` resource:

```text
GET    /users
GET    /users/10
POST   /users
PUT    /users/10
PATCH  /users/10
DELETE /users/10
```

---

# 6. GET

Used to retrieve information.

Example:

```http
GET /users/10
```

Response:

```json
{
  "id": 10,
  "name": "Sekhar"
}
```

GET requests should generally not modify server-side data.

---

# 7. POST

Used to submit data, commonly to create a new resource.

Request:

```http
POST /users
```

Body:

```json
{
  "name": "Sekhar",
  "email": "sekhar@example.com"
}
```

The server may create a new user and return the created resource.

---

# 8. PUT

Used to replace/update a resource.

Example:

```http
PUT /users/10
```

Body:

```json
{
  "name": "Sekhar",
  "email": "new@example.com"
}
```

PUT generally represents replacing the resource representation.

---

# 9. PATCH

Used for a partial update.

Example:

```http
PATCH /users/10
```

Body:

```json
{
  "email": "new@example.com"
}
```

Only the specified field needs to be changed.

---

# 10. DELETE

Used to delete a resource.

```http
DELETE /users/10
```

The server deletes user `10`.

---

# 11. HTTP Request

An HTTP request can contain several important parts:

```text
HTTP Method
URL
Headers
Request Body
```

Example:

```http
POST /users
Content-Type: application/json
Authorization: Bearer <token>

{
  "name": "Sekhar"
}
```

---

# 12. HTTP Response

An HTTP response generally contains:

```text
Status Code
Headers
Response Body
```

Example:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 10,
  "name": "Sekhar"
}
```

---

# 13. HTTP Status Codes

You should know the major categories.

### 2xx — Success

```text
200 OK
201 Created
204 No Content
```

Common examples:

- `200` → request successful
- `201` → resource successfully created
- `204` → successful request with no response body

### 4xx — Client Error

```text
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
```

Examples:

- `400` → invalid request
- `401` → authentication required/failed
- `403` → authenticated but not allowed
- `404` → resource not found
- `409` → conflict with current resource state

### 5xx — Server Error

```text
500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
```

Generally indicates a problem while processing the request on the server or an upstream service.

---

# 14. Path Variable

A **path variable** identifies a specific resource.

Example:

```text
GET /users/10
```

Here:

```text
10
```

is the user ID.

Conceptually:

```text
/users/{id}
```

In Spring Boot:

```java
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) {
    ...
}
```

---

# 15. Query Parameter

Query parameters are commonly used for filtering, searching, sorting, or pagination.

Example:

```text
GET /users?city=bangalore
```

Multiple parameters:

```text
GET /users?city=bangalore&page=1&size=20
```

Conceptually:

```text
/users?key=value
```

Difference:

```text
/users/10
       ↑
Path variable → identifies the resource

/users?city=bangalore
       ↑
Query parameter → filters/searches/customizes the request
```

---

# 16. Request Body

The request body contains data sent by the client.

Example:

```http
POST /transactions
```

Body:

```json
{
  "amount": 500,
  "category": "Food",
  "type": "expense"
}
```

For REST APIs, JSON is the most common format.

---

# 17. JSON

**JSON (JavaScript Object Notation)** is a lightweight data-interchange format.

Example:

```json
{
  "id": 101,
  "name": "Sekhar",
  "active": true
}
```

Nested data:

```json
{
  "id": 101,
  "customer": {
    "name": "Sekhar"
  }
}
```

Arrays:

```json
{
  "users": [
    {
      "id": 1,
      "name": "A"
    },
    {
      "id": 2,
      "name": "B"
    }
  ]
}
```

Spring Boot can automatically convert between Java objects and JSON using its HTTP message conversion infrastructure.

---

# 18. Headers

HTTP headers provide additional information about the request or response.

Common headers:

```text
Content-Type
Accept
Authorization
Cache-Control
```

### Content-Type

Specifies the format of the request/response body.

Example:

```text
Content-Type: application/json
```

### Accept

Tells the server what response formats the client can accept.

Example:

```text
Accept: application/json
```

### Authorization

Used to send authentication credentials/token information.

Example:

```text
Authorization: Bearer <token>
```

---

# 19. REST Constraints

REST is an architectural style with several constraints.

The important ones for interviews are:

### Client-Server

Client and server have separate responsibilities.

```text
React → UI
Backend → Business logic/data
```

### Stateless

Each request should contain the information necessary for the server to process it.

The server should not depend on remembering the client's previous request as conversational state.

For example:

```text
Request 1
Request 2
Request 3
```

Each request should be independently understandable by the server.

Authentication tokens are commonly sent with each request for this reason.

### Cacheable

Responses can indicate whether they may be cached.

### Uniform Interface

REST APIs follow consistent conventions for interacting with resources.

### Layered System

A client does not necessarily know whether it is communicating directly with the actual server or through intermediaries such as proxies or gateways.

You don't need to memorize every REST constraint immediately. Understand the concepts.

---

# 20. Statelessness

This is particularly important for interviews.

Suppose a client sends:

```text
Request 1 → Login
Request 2 → Get transactions
Request 3 → Create transaction
```

A stateless API doesn't require the server to remember the complete conversational state from Request 1 in order to understand Request 2.

For authenticated APIs, the client commonly sends an authentication token with each request:

```text
Authorization: Bearer <token>
```

This concept becomes important later when you learn **JWT and Spring Security**.

---

# 21. Idempotency

An operation is **idempotent** if making the same request multiple times has the same intended effect on the server state as making it once.

Commonly:

```text
GET     → idempotent
PUT     → idempotent
DELETE  → idempotent
POST    → generally not idempotent
```

Example:

```text
PUT /users/10
```

with the same data multiple times should leave user 10 in the same state.

But:

```text
POST /orders
```

may create a new order every time it is sent.

This becomes important in distributed systems and retries.

---

# 22. REST vs SOAP

You may encounter this in interviews.

### REST

- Architectural style
- Commonly uses HTTP
- Often uses JSON
- Lightweight and widely used for modern web/mobile applications
- Uses HTTP methods naturally

### SOAP

- Protocol
- Uses XML-based messages
- Has formal standards/specifications
- Common in some enterprise/integration systems

For your current preparation, **understand the difference rather than going deep into SOAP**.

---

# 23. REST API Example — Finance Tracker

Your Finance Tracker could expose APIs like:

```text
GET    /transactions
GET    /transactions/123
POST   /transactions
PUT    /transactions/123
DELETE /transactions/123
```

Example POST:

```http
POST /transactions
Content-Type: application/json
```

```json
{
  "amount": 500,
  "category": "Food",
  "type": "expense",
  "description": "Lunch"
}
```

Possible response:

```http
201 Created
```

```json
{
  "id": 123,
  "amount": 500,
  "category": "Food",
  "type": "expense",
  "description": "Lunch"
}
```

This is essentially what you'll later implement with Spring Boot.

---

# 24. REST API Architecture

A typical backend can be structured as:

```text
React Frontend
      ↓
REST API
      ↓
Controller
      ↓
Service
      ↓
Repository
      ↓
Database
```

For example:

```text
POST /transactions
       ↓
TransactionController
       ↓
TransactionService
       ↓
TransactionRepository
       ↓
MySQL
```

You'll learn how Spring Boot implements these layers later.

---

# 25. REST API vs Web Service

The terms are related but not identical.

A **web service** is a broad concept for providing functionality over a network.

A **REST API** is one way of designing a network API using REST principles.

So:

```text
Web Service
   ├── REST
   └── SOAP
```

This is a simplified view, but useful for understanding the terminology.

---

# 26. Important Interview Questions

### What is REST?

REST is an architectural style for designing networked applications around resources and a uniform interface, commonly using HTTP.

### What is a REST API?

An API designed according to REST principles, commonly using HTTP methods, resource-oriented URLs, status codes, and representations such as JSON.

### What is the difference between PUT and PATCH?

PUT generally replaces the resource representation, while PATCH performs a partial modification.

### What is the difference between path parameters and query parameters?

Path parameters typically identify a specific resource, while query parameters commonly filter, search, sort, or paginate a collection.

### What does stateless mean in REST?

Each request contains the information necessary for the server to process it; the server does not rely on conversational state from previous requests.

### What is the difference between 401 and 403?

`401 Unauthorized` generally means authentication is required or failed.

`403 Forbidden` means the server understood the request but refuses to authorize the operation.

### What is the difference between 200 and 201?

`200 OK` indicates successful processing.

`201 Created` indicates that a new resource was successfully created.

### What is JSON?

A lightweight text-based data-interchange format commonly used to exchange data between clients and REST APIs.

### What is idempotency?

An operation is idempotent when repeating the same request has the same intended effect on server state as performing it once.

---

# What You Need to Remember

For your current preparation, the core mental model is:

```text
Client
  ↓
HTTP Request
  ↓
REST API
  ↓
Backend
  ↓
Database
  ↓
HTTP Response
  ↓
Client
```

And know these well:

```text
GET
POST
PUT
PATCH
DELETE

200
201
204
400
401
403
404
409
500

Path Variable
Query Parameter
Request Body
Headers
JSON
Statelessness
Idempotency
REST vs SOAP
```
