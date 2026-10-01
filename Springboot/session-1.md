# Spring Boot — Web, REST API and Spring MVC

## 1. How a Java Web Application Handles HTTP Requests

A client communicates with a server using the **HTTP protocol**.

Basic flow:

```text
Client
   ↓ HTTP Request
Web Server / Servlet Container
   ↓
Servlet
   ↓
Application Code
   ↓
HTTP Response
   ↓
Client
```

### Servlet

A **Servlet** is a Java component that handles HTTP requests and produces HTTP responses.

Servlets are executed inside a **Servlet Container**.

### Servlet Container

A Servlet Container is responsible for:

- Creating and managing servlet objects
- Receiving HTTP requests
- Passing requests to the appropriate servlet
- Managing servlet lifecycle
- Sending servlet responses back to the client

**Apache Tomcat** is a commonly used Servlet Container.

---

# 2. Spring Boot and Tomcat

When building a traditional Java web application, we may deploy the application to an externally installed Tomcat server.

Spring Boot simplifies this by providing an **embedded web server**.

For a typical Spring Boot web application using Spring MVC:

```text
Spring Boot Application
        ↓
Embedded Tomcat
        ↓
Servlet Container
        ↓
Spring MVC
        ↓
Controllers
```

The Tomcat server is started as part of the Spring Boot application.

Therefore, we normally do not need to separately install and deploy the application to Tomcat.

The default port is:

```text
http://localhost:8080
```

The port can be changed through application configuration.

---

# 3. Spring Web Dependency

For a Spring Boot application that handles web requests using Spring MVC, we commonly add:

```text
Spring Web
```

This provides the dependencies required for building web applications and REST APIs with Spring MVC.

It brings in components such as:

- Spring MVC
- Embedded web server support
- HTTP request/response handling
- REST-related functionality

---

# 4. What Happens When a Spring Boot Application Starts?

When we run:

```java
SpringApplication.run(SimpleWebAppApplication.class, args);
```

Spring Boot:

1. Creates the Spring Application Context.
2. Performs component scanning and auto-configuration.
3. Creates the required Spring-managed objects (beans).
4. Starts the embedded web server.
5. The application begins listening for HTTP requests.

For example:

```text
Application started
       ↓
Embedded Tomcat started
       ↓
Listening on port 8080
       ↓
Waiting for HTTP requests
```

At this point, no request needs to have been sent yet.

---

# 5. Controllers

In Spring MVC, controllers are responsible for handling incoming HTTP requests.

A controller can be created using a Java class annotated with:

```java
@Controller
```

or:

```java
@RestController
```

Spring detects these classes through component scanning and registers their request mappings.

---

# 6. @Controller

`@Controller` is used for Spring MVC controllers that commonly return **view names**.

Example:

```java
@Controller
public class HomeController {

    @RequestMapping("/")
    public String home() {
        return "home";
    }
}
```

Here:

```java
return "home";
```

is interpreted as a **view name**, not directly as the response body.

Spring MVC attempts to resolve a view named `home`.

If no corresponding view is configured, the request may result in an error such as a view/resource resolution error.

---

# 7. @ResponseBody

`@ResponseBody` tells Spring:

> Treat the return value of this method as the HTTP response body instead of treating it as a view name.

Example:

```java
@Controller
public class HomeController {

    @RequestMapping("/")
    @ResponseBody
    public String home() {
        return "This is Home page";
    }
}
```

Now:

```text
"This is Home page"
```

is sent directly as the HTTP response body.

---

# 8. @RestController

`@RestController` is commonly used when building REST APIs.

It is effectively a combination of:

```java
@Controller
@ResponseBody
```

Therefore:

```java
@RestController
public class HomeController {

    @RequestMapping("/")
    public String home() {
        return "This is Home page";
    }
}
```

The returned string is treated as response data rather than a view name.

### Important

`@RestController` does **not** mean that the method must return JSON.

It can return:

- String
- Object
- List
- Other response data

When an object is returned, Spring can serialize it into JSON using an HTTP message converter, commonly backed by **Jackson**.

---

# 9. Request Mapping

`@RequestMapping` maps an HTTP request to a controller class or controller method.

Example:

```java
@RequestMapping("/")
public String home() {
    return "This is Home page";
}
```

A request such as:

```text
GET /
```

can be mapped to this method.

We can also create multiple mappings:

```java
@RestController
public class HomeController {

    @RequestMapping("/")
    public String home() {
        return "This is Home page";
    }

    @RequestMapping("/about")
    public String about() {
        return "This is about page";
    }
}
```

---

# 10. How Does Spring Know Which Controller Should Handle a Request?

Spring MVC uses a **Front Controller** architecture.

The central component is:

```text
DispatcherServlet
```

The basic flow is:

```text
Client
   ↓
HTTP Request
   ↓
Embedded Tomcat
   ↓
DispatcherServlet
   ↓
Find matching Controller + Handler Method
   ↓
Controller Method
   ↓
Response
```

The `DispatcherServlet` acts as the central entry point for requests in Spring MVC.

It uses the configured request mappings to determine which controller method should handle a request.

It does not simply "redirect" the request to a controller. More accurately, it **dispatches the request** to the appropriate handler method.

---

# 11. REST API

A REST API allows clients and servers to communicate using HTTP.

Common HTTP methods include:

| HTTP Method | Common Purpose        |
| ----------- | --------------------- |
| GET         | Retrieve data         |
| POST        | Create data           |
| PUT         | Update/replace data   |
| PATCH       | Partially update data |
| DELETE      | Delete data           |

For example:

```text
GET    /products
GET    /products/101
POST   /products
PUT    /products/101
DELETE /products/101
```

Tools such as Postman can be used to send and test these HTTP requests.

---

# 12. Path Variables

Suppose we want to retrieve a product using its ID.

A URL could be:

```text
/products/103
```

The `103` is part of the URL path.

We can capture this value using:

```java
@GetMapping("/products/{prodId}")
public Product getProductById(@PathVariable int prodId) {
    return service.getProductById(prodId);
}
```

Here:

```text
/products/{prodId}
```

defines a path variable.

If the request is:

```text
/products/103
```

then:

```java
prodId
```

will receive:

```text
103
```

### `@PathVariable`

`@PathVariable` tells Spring to obtain the value from the URL path.

---

# 13. @GetMapping, @PostMapping, @PutMapping and @DeleteMapping

Instead of using `@RequestMapping` for every HTTP method, Spring provides specialized annotations.

```java
@GetMapping("/products")
```

```java
@PostMapping("/products")
```

```java
@PutMapping("/products")
```

```java
@DeleteMapping("/products/{prodId}")
```

These make the intended HTTP method explicit.

### Example

```java
@GetMapping("/products")
public List<Product> getProducts() {
    return service.getProducts();
}
```

A GET request to:

```text
/products
```

will invoke this method.

---

# 14. @RequestBody

When the client sends data in the HTTP request body, we can use:

```java
@RequestBody
```

Example:

```java
@PostMapping("/products")
public void addProduct(@RequestBody Product prod) {
    service.addProduct(prod);
}
```

Suppose the client sends JSON:

```json
{
  "prodId": 104,
  "prodName": "OnePlus",
  "prodPrice": 30000
}
```

Spring reads the JSON request body and converts it into a `Product` object.

Conceptually:

```text
JSON
 ↓
HTTP Message Converter
 ↓
Product Java Object
```

Jackson is commonly used for this JSON serialization/deserialization.

---

# 15. JSON and Jackson

**Jackson** is commonly used by Spring Boot for converting between Java objects and JSON.

### Java Object → JSON

This is called **serialization**.

```text
Product object
      ↓
   Jackson
      ↓
JSON
```

### JSON → Java Object

This is called **deserialization**.

```text
JSON
 ↓
Jackson
 ↓
Product object
```

For example:

```java
Product product
```

can be converted to:

```json
{
  "prodId": 101,
  "prodName": "IPhone",
  "prodPrice": 50000
}
```

---

# 16. Service Layer

The controller should generally handle HTTP-related responsibilities rather than containing all business logic.

A common structure is:

```text
Controller
    ↓
Service
    ↓
Repository / Database
```

Example:

```java
@RestController
public class ProductController {

    @Autowired
    ProductService service;

    @GetMapping("/products")
    public List<Product> getProducts() {
        return service.getProducts();
    }
}
```

The controller receives the HTTP request and delegates the operation to:

```java
ProductService
```

---

# 17. @Service

The `@Service` annotation identifies a class as a Spring-managed service component.

Example:

```java
@Service
public class ProductService {

}
```

During component scanning, Spring detects the class and creates a bean for it.

That bean can then be injected into another Spring-managed class.

---

# 18. Dependency Injection

Instead of manually creating the service:

```java
ProductService service = new ProductService();
```

we allow Spring to manage the object.

Example:

```java
@Autowired
ProductService service;
```

Spring finds the `ProductService` bean and injects it into the controller.

Conceptually:

```text
Spring Container
      │
      ├── ProductController
      │
      └── ProductService
              ↑
              │
       dependency injected
```

### Constructor Injection

Constructor injection is generally preferred in modern Spring applications:

```java
@RestController
public class ProductController {

    private final ProductService service;

    public ProductController(ProductService service) {
        this.service = service;
    }
}
```

---

# 19. Product REST API Example

## Controller

```java
@RestController
public class ProductController {

    @Autowired
    ProductService service;

    @GetMapping("/products")
    public List<Product> getProducts() {
        return service.getProducts();
    }

    @GetMapping("/products/{prodId}")
    public Product getProductById(@PathVariable int prodId) {
        return service.getProductById(prodId);
    }

    @PostMapping("/products")
    public void addProduct(@RequestBody Product prod) {
        service.addProduct(prod);
    }

    @PutMapping("/products")
    public void updateProduct(@RequestBody Product prod) {
        service.updateProduct(prod);
    }

    @DeleteMapping("/products/{prodId}")
    public void deleteProductById(@PathVariable int prodId) {
        service.deleteProductById(prodId);
    }
}
```

---

# 20. Product Model

```java
public class Product {

    private int prodId;
    private String prodName;
    private int prodPrice;

    public Product(int prodId, String prodName, int prodPrice) {
        this.prodId = prodId;
        this.prodName = prodName;
        this.prodPrice = prodPrice;
    }

    public int getProdId() {
        return prodId;
    }

    public void setProdId(int prodId) {
        this.prodId = prodId;
    }

    public String getProdName() {
        return prodName;
    }

    public void setProdName(String prodName) {
        this.prodName = prodName;
    }

    public int getProdPrice() {
        return prodPrice;
    }

    public void setProdPrice(int prodPrice) {
        this.prodPrice = prodPrice;
    }
}
```

---

# 21. Product Service

```java
@Service
public class ProductService {

    List<Product> products = new ArrayList<>(
        Arrays.asList(
            new Product(101, "IPhone", 50000),
            new Product(102, "Samsung", 20000),
            new Product(103, "Realme", 10000)
        )
    );

    public List<Product> getProducts() {
        return products;
    }

    public Product getProductById(int id) {
        return products.stream()
                .filter(p -> p.getProdId() == id)
                .findFirst()
                .orElse(new Product(0, "No Item", 0));
    }

    public void addProduct(Product product) {
        products.add(product);
    }

    public void updateProduct(Product prod) {
        for (int i = 0; i < products.size(); i++) {
            if (products.get(i).getProdId() == prod.getProdId()) {
                products.set(i, prod);
                return;
            }
        }
    }

    public void deleteProductById(int prodId) {
        for (int i = 0; i < products.size(); i++) {
            if (products.get(i).getProdId() == prodId) {
                products.remove(i);
                return;
            }
        }
    }
}
```

For a real application, this data would normally be stored in a database rather than an in-memory `List`.

---

# 22. Complete Request Flow

Consider:

```text
GET /products/103
```

The request flows approximately like this:

```text
Client / Postman
      ↓
HTTP GET /products/103
      ↓
Embedded Tomcat
      ↓
DispatcherServlet
      ↓
Spring MVC finds:
@GetMapping("/products/{prodId}")
      ↓
ProductController
      ↓
@PathVariable extracts 103
      ↓
ProductService
      ↓
Find product with ID 103
      ↓
Product object returned
      ↓
Jackson converts Product → JSON
      ↓
HTTP Response
      ↓
Client
```

Example response:

```json
{
  "prodId": 103,
  "prodName": "Realme",
  "prodPrice": 10000
}
```

---

# 23. Overall Spring Boot REST Architecture

```text
                    Client
                      │
                      │ HTTP Request
                      ↓
              Embedded Tomcat
                      │
                      ↓
              DispatcherServlet
                      │
                      ↓
                 Controller
                      │
                      ↓
                  Service
                      │
                      ↓
             Repository / DAO
                      │
                      ↓
                   Database
```

For a simple application without a database:

```text
Client
  ↓
Tomcat
  ↓
DispatcherServlet
  ↓
Controller
  ↓
Service
  ↓
In-memory List
```

---

# 24. Important Annotations

| Annotation               | Purpose                                        |
| ------------------------ | ---------------------------------------------- |
| `@SpringBootApplication` | Main Spring Boot application configuration     |
| `@Controller`            | Defines an MVC controller, commonly for views  |
| `@RestController`        | Controller whose methods return response data  |
| `@RequestMapping`        | Maps requests to controller methods/classes    |
| `@GetMapping`            | Maps HTTP GET requests                         |
| `@PostMapping`           | Maps HTTP POST requests                        |
| `@PutMapping`            | Maps HTTP PUT requests                         |
| `@DeleteMapping`         | Maps HTTP DELETE requests                      |
| `@PathVariable`          | Reads a value from the URL path                |
| `@RequestBody`           | Reads data from the HTTP request body          |
| `@Service`               | Marks a service class as a Spring-managed bean |
| `@Autowired`             | Injects a Spring-managed dependency            |

---

# 25. Key Concepts to Remember

### Traditional Java Web Application

```text
HTTP Request
     ↓
Tomcat / Servlet Container
     ↓
Servlet
     ↓
Application
```

### Spring Boot Web Application

```text
HTTP Request
     ↓
Embedded Tomcat
     ↓
DispatcherServlet
     ↓
Controller
     ↓
Service
     ↓
Repository / Database
```

Spring MVC still operates on top of the **Servlet API**. Spring Boot does not replace the underlying servlet-based web infrastructure when using the traditional Spring MVC stack; it simplifies configuration and application startup.

### Important distinction

```text
Tomcat
    → Servlet Container

Servlet
    → Java component that handles requests

DispatcherServlet
    → Spring MVC's Front Controller

Controller
    → Application-level request handler

Service
    → Business logic

Jackson
    → JSON ↔ Java object conversion
```

---

# 26. REST CRUD Example

A typical Product REST API can expose:

```text
GET    /products
       → Get all products

GET    /products/{id}
       → Get one product

POST   /products
       → Create a product

PUT    /products
       → Update a product

DELETE /products/{id}
       → Delete a product
```

This gives the basic CRUD operations:

```text
Create → POST
Read   → GET
Update → PUT
Delete → DELETE
```
