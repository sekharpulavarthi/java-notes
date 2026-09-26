# Session 1: Servlets

## 1. What is a Servlet?

A Servlet is a Java class that runs on a web server and handles HTTP requests and responses.

It is commonly used to build server-side Java web applications.

Basic flow:

```text
Client
  ↓ HTTP Request
Web Server / Servlet Container (Tomcat)
  ↓
Servlet
  ↓
Application Logic
  ↓
HTTP Response
  ↓
Client
```

A Servlet itself does not run directly like a normal Java `main()` program. It runs inside a **Servlet Container**.

---

## 2. What is Tomcat?

Apache Tomcat is a **Servlet Container**.

It:

- Starts and manages Servlets
- Creates Servlet objects
- Calls their lifecycle methods
- Receives HTTP requests
- Sends requests to the appropriate Servlet
- Sends the Servlet's response back to the client

So:

```text
Tomcat = Runtime environment for Servlets
```

Tomcat also acts as a web server for Java web applications.

---

## 3. What is a Servlet Container?

A Servlet Container manages the lifecycle of Servlets.

Its responsibilities include:

- Creating Servlet instances
- Calling lifecycle methods
- Mapping URLs to Servlets
- Providing request/response objects
- Managing multiple requests
- Handling Servlet lifecycle

Tomcat is an example of a Servlet Container.

---

## 4. HttpServlet

Most HTTP Servlets extend:

```java
HttpServlet
```

Example:

```java
public class HelloServlet extends HttpServlet {

    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response) {

        // handle request
    }
}
```

`HttpServlet` provides methods for handling different HTTP operations.

Common methods:

```text
doGet()
doPost()
doPut()
doDelete()
```

For your interview preparation, `doGet()` and `doPost()` are the most important to understand.

---

## 5. GET and POST

### GET

Used mainly for retrieving data.

Example:

```text
GET /users/10
```

The server receives a request asking for user 10.

### POST

Used mainly for sending data to the server, often to create something.

Example:

```text
POST /users
```

with request data:

```json
{
  "name": "Sekhar"
}
```

Basic idea:

```text
GET  → retrieve
POST → send/create
```

Don't treat this as an absolute rule; HTTP semantics are more precise than simply "GET = fetch, POST = create."

---

## 6. HttpServletRequest

`HttpServletRequest` represents the incoming HTTP request.

It allows the Servlet to access things such as:

- Request parameters
- Headers
- HTTP method
- Request URI
- Request body
- Cookies

Example:

```java
String name = request.getParameter("name");
```

---

## 7. HttpServletResponse

`HttpServletResponse` represents the response that the server sends back to the client.

It can be used to set:

- Status code
- Response headers
- Response body
- Content type

Conceptually:

```text
Request  → HttpServletRequest
Response → HttpServletResponse
```

---

## 8. Servlet Lifecycle

The Servlet Container manages the Servlet lifecycle.

The important methods are:

```text
init()
  ↓
service()
  ↓
destroy()
```

### init()

Called when the Servlet is initialized.

Used for initialization work.

### service()

Called when a request arrives.

For `HttpServlet`, the `service()` method determines the HTTP method and eventually invokes methods such as:

```text
GET    → doGet()
POST   → doPost()
PUT    → doPut()
DELETE → doDelete()
```

### destroy()

Called when the Servlet is being removed/shut down.

---

## 9. URL Mapping

The container needs to know which Servlet should handle a particular URL.

For example:

```text
/users
```

could be mapped to:

```text
UserServlet
```

Modern Servlet applications can use annotations such as:

```java
@WebServlet("/users")
public class UserServlet extends HttpServlet {
}
```

Then:

```text
GET /users
       ↓
UserServlet
```

---

## 10. Servlet Instance and Multiple Requests

The container generally creates a Servlet instance and uses it to handle requests.

Therefore, Servlets should be designed carefully with respect to **shared instance state**.

Avoid storing request-specific data in instance variables.

For example, this is dangerous:

```java
private String username;
```

if different requests can modify it.

Prefer local variables inside request-handling methods.

---

## 11. Important Interview Questions

### What is a Servlet?

A Java class that handles HTTP requests and generates HTTP responses, running inside a Servlet Container.

### What is Tomcat?

A Servlet Container that manages and runs Java Servlets.

### What is the Servlet lifecycle?

```text
init()
service()
destroy()
```

### What is the difference between `doGet()` and `doPost()`?

`doGet()` handles GET requests, while `doPost()` handles POST requests.

### What are HttpServletRequest and HttpServletResponse?

`HttpServletRequest` represents the incoming request, while `HttpServletResponse` represents the outgoing response.

### Why do we need Tomcat?

A Servlet requires a Servlet Container to be created, managed, and executed. Tomcat provides that environment.
