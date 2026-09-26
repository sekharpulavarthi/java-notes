# Session 2: JSP and the Connection to Spring

## 1. What is JSP?

JSP stands for **JavaServer Pages**.

It was designed to make it easier to generate dynamic HTML from Java-based web applications.

Instead of writing HTML entirely inside Java code, developers could create pages containing HTML with server-side Java/JSP constructs.

Conceptually:

```text
Browser
   ↓
Request
   ↓
Servlet
   ↓
JSP
   ↓
HTML Response
   ↓
Browser
```

---

## 2. Why Was JSP Used?

Early Java web applications commonly used:

```text
Servlets → Business Logic
JSP     → Presentation/UI
```

The Servlet could process the request and forward the result to a JSP page.

The JSP would generate HTML.

This was one way of implementing an MVC-style application.

---

## 3. JSP vs Servlet

### Servlet

Primarily Java code:

```java
public class UserServlet extends HttpServlet {
    ...
}
```

Good for handling requests and application flow, but generating large amounts of HTML directly in Java becomes difficult to maintain.

### JSP

Primarily HTML with server-side Java/JSP features.

Better suited to generating server-rendered HTML.

So historically:

```text
Servlet → Controller
JSP     → View
```

---

## 4. JSP Lifecycle

You don't need to memorize the complete JSP lifecycle.

The important concept is:

**A JSP is ultimately processed by the Servlet infrastructure.**

The JSP container translates/compiles the JSP into a Servlet, which can then execute.

Therefore, you can think of:

```text
JSP
 ↓
Servlet
 ↓
HTTP Response
```

This is enough for your current level.

---

## 5. JSP Today

For your career path, JSP is **not something you need to practice**.

You are targeting:

```text
React
   ↓
REST API
   ↓
Spring Boot
   ↓
JPA / Hibernate
   ↓
Database
```

This is fundamentally different from building a traditional server-rendered application using JSP.

Therefore, don't spend time learning JSP syntax or building JSP applications.

---

# 6. Servlet → Spring Connection

This is the most useful part for you.

Spring MVC/Spring Boot does not completely replace the underlying Java web infrastructure.

At a high level:

```text
Client
   ↓
HTTP Request
   ↓
Tomcat / Servlet Container
   ↓
Spring's web infrastructure
   ↓
Controller
   ↓
Service
   ↓
Repository
   ↓
Database
   ↓
Response
```

A Spring Boot application can run with an embedded server such as Tomcat.

So when you write:

```java
@RestController
public class UserController {

    @GetMapping("/users")
    public List<User> getUsers() {
        ...
    }
}
```

you don't manually create a Servlet and implement `doGet()`.

Spring handles much of that infrastructure for you.

---

## 7. Traditional Servlet vs Spring MVC

Traditional Servlet:

```java
public class UserServlet extends HttpServlet {

    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response) {

        // manually handle request
    }
}
```

Spring MVC:

```java
@RestController
public class UserController {

    @GetMapping("/users")
    public List<User> getUsers() {
        ...
    }
}
```

Spring provides abstractions that make web application development much easier.

You therefore don't need to manually manage all the Servlet infrastructure in a normal Spring Boot application.

---

# 8. Important Terms to Remember

Remember these relationships:

```text
HTTP
 ↓
Servlet
 ↓
Servlet Container
 ↓
Tomcat
```

and:

```text
Spring Boot
 ↓
Spring MVC
 ↓
Servlet infrastructure
 ↓
Tomcat
```

For your current preparation, you don't need to memorize every internal implementation detail.

---

# 9. What You Should Know for Interviews

Be able to answer:

### What is a Servlet?

A Java class that handles HTTP requests and responses inside a Servlet Container.

### What is Tomcat?

A Servlet Container used to run Java web applications and manage Servlets.

### What is JSP?

A server-side view technology historically used to generate dynamic HTML in Java web applications.

### Servlet vs JSP?

Servlets primarily handle request processing/application flow, while JSP was primarily used for the presentation/view layer.

### Why don't we normally write Servlets directly in Spring Boot?

Spring provides higher-level abstractions such as Spring MVC controllers, so developers don't need to manually implement the Servlet request-handling infrastructure.

### Does Spring Boot use Tomcat?

Spring Boot commonly uses embedded Tomcat as its default servlet container for traditional Spring MVC web applications, though other servers can be configured.

---

# Final Takeaway

For your current preparation:

```text
Servlets → Understand
Tomcat → Understand what it does
HttpServlet → Understand
Request/Response → Understand
Servlet lifecycle → Understand
GET/POST → Understand
JSP → Understand what it is
JSP syntax → Skip
Servlet coding → Skip
Tomcat setup → Skip
Java EE setup → Skip
```

Then move directly to:

```text
REST API
   ↓
ORM
   ↓
Hibernate
   ↓
Spring
   ↓
Spring Boot
   ↓
Spring Data JPA
   ↓
Spring MVC
```
