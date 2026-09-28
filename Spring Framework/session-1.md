# Spring Framework — IoC, Dependency Injection & Autowiring

## 1. What is Spring?

Spring is a Java framework used to build **enterprise-level applications**.

It can also be used to build **web applications**.

A typical application can be organized into layers:

```text
Controller
    ↓
Service
    ↓
Repository
```

- **Controller** — handles requests and responses.
- **Service** — contains business logic.
- **Repository** — handles data/database operations.

---

# 2. The Problem Spring Solves

In normal Java, if one class depends on another class, we can create the dependency ourselves.

For example:

```java
class Controller {

    Service service = new Service();

}
```

Similarly:

```java
class Service {

    Repository repository = new Repository();

}
```

For a small application this is manageable.

But as an application grows, there can be hundreds or thousands of objects and dependencies.

The application then has to manage:

- Object creation
- Which objects depend on which
- Object lifecycle
- Object destruction

This creates tighter coupling between classes.

Spring solves this by taking responsibility for managing application objects.

The basic idea is:

> **"You focus on your business logic; Spring manages the objects and their dependencies."**

---

# 3. Inversion of Control (IoC)

**Inversion of Control (IoC)** means that the responsibility for creating and managing objects is transferred from our application code to the Spring container.

Normally:

```text
Application code
      ↓
creates objects
      ↓
manages dependencies
```

With Spring:

```text
Spring Container
      ↓
creates objects
      ↓
manages dependencies
      ↓
provides objects to our application
```

The control over object creation has been **inverted**.

---

# 4. IoC Container

Spring provides an **IoC container** that manages Spring objects.

The container is responsible for:

- Creating objects (beans)
- Managing their lifecycle
- Resolving dependencies
- Injecting dependencies

Objects managed by Spring are called **Spring Beans**.

The container is ultimately represented through interfaces such as:

```java
ApplicationContext
```

Example:

```java
ApplicationContext context =
        SpringApplication.run(MyAppApplication.class, args);
```

`context` gives us access to the Spring container.

We can ask the container for a bean:

```java
Dev obj = context.getBean(Dev.class);

obj.build();
```

---

# 5. Does Spring Create Objects for Every Class?

**No.**

Spring does not automatically create objects for every Java class in the project.

Spring creates and manages objects for classes that it knows should be Spring beans.

For example:

```java
@Component
public class Dev {
}
```

Now Spring knows:

> `Dev` is a class that I should manage.

Therefore, Spring creates a `Dev` object and stores/manages it as a bean inside the IoC container.

### Important correction

The JVM itself does **not** contain an "IoC container."

The Spring IoC container is a Spring-managed object/system that itself runs inside the JVM.

Think of it as:

```text
JVM
 └── Spring Application
      └── IoC Container / ApplicationContext
           ├── Dev bean
           ├── Laptop bean
           ├── Service bean
           └── Repository bean
```

---

# 6. Container Before Objects

When a Spring application starts, the Spring container is initialized.

The container then identifies the classes that should be managed and creates their beans according to Spring's configuration/component scanning and bean lifecycle rules.

So conceptually:

```text
Start application
       ↓
Create/initialize Spring container
       ↓
Find Spring-managed components
       ↓
Create required beans
       ↓
Resolve dependencies
       ↓
Application is ready
```

It is therefore useful to think:

> **The container manages the beans; the beans are not simply "stored in the JVM."**

---

# 7. SpringApplication.run()

In Spring Boot:

```java
ApplicationContext context =
        SpringApplication.run(MyAppApplication.class, args);
```

`SpringApplication.run()` starts the Spring application and returns an `ApplicationContext`.

Through that context, we can access Spring-managed beans:

```java
Dev obj = context.getBean(Dev.class);

obj.build();
```

So:

```text
SpringApplication.run()
        ↓
starts Spring application
        ↓
creates/initializes ApplicationContext
        ↓
Spring manages beans inside the context
        ↓
ApplicationContext can provide beans
```

---

# 8. Dependency Injection (DI)

**Dependency Injection is a way of implementing IoC.**

A dependency is an object that another class needs.

Example:

```java
class Dev {

    private Laptop laptop;

}
```

`Dev` depends on `Laptop`.

Instead of `Dev` creating the dependency itself:

```java
Laptop laptop = new Laptop();
```

Spring can provide the `Laptop` object.

That is **Dependency Injection**.

```text
Spring Container
      ↓
creates Laptop
      ↓
injects Laptop into Dev
```

This reduces coupling between classes.

---

# 9. Types of Dependency Injection

There are three commonly discussed forms:

### 1. Constructor Injection

```java
private Laptop laptop;

public Dev(Laptop laptop) {
    this.laptop = laptop;
}
```

Constructor injection is generally preferred.

With a single constructor, Spring can perform constructor injection without requiring `@Autowired`.

---

### 2. Setter Injection

```java
private Laptop laptop;

@Autowired
public void setLaptop(Laptop laptop) {
    this.laptop = laptop;
}
```

Spring calls the setter and provides the dependency.

---

### 3. Field Injection

```java
@Autowired
private Laptop laptop;
```

This is called field injection.

It is generally **not recommended** compared with constructor injection.

---

# 10. Why Dependency Injection?

DI helps achieve **loose coupling**.

Without DI:

```java
class Dev {

    Laptop laptop = new Laptop();

}
```

`Dev` directly creates its dependency.

With DI:

```java
class Dev {

    private Laptop laptop;

    public Dev(Laptop laptop) {
        this.laptop = laptop;
    }

}
```

Now `Dev` doesn't need to know how `Laptop` is created.

Spring provides it.

---

# 11. @Component

Spring Boot makes it easy to tell Spring which classes it should manage.

Example:

```java
@Component
public class Dev {
}
```

and:

```java
@Component
public class Laptop {
}
```

`@Component` tells Spring:

> **"Manage this class as a Spring bean."**

Spring's component scanning finds these classes and registers them as beans.

---

# 12. Autowiring

Suppose we have:

```java
@Component
public class Dev {

    private Laptop laptop;

    public void build() {
        laptop.compile();
        System.out.println("Working on spring project");
    }
}
```

and:

```java
@Component
public class Laptop {

    public void compile() {
        System.out.println("This is laptop class");
    }
}
```

If `laptop` is never initialized, this:

```java
laptop.compile();
```

would cause a `NullPointerException`.

Spring can inject the `Laptop` dependency automatically.

Using constructor injection:

```java
@Component
public class Dev {

    private Laptop laptop;

    public Dev(Laptop laptop) {
        this.laptop = laptop;
    }

    public void build() {
        laptop.compile();
        System.out.println("Working on spring project");
    }
}
```

Spring sees that `Dev` requires a `Laptop`.

Because `Laptop` is also a Spring bean:

```java
@Component
public class Laptop {
}
```

Spring provides the `Laptop` bean to the `Dev` constructor.

---

# 13. Autowiring by Type

Spring generally resolves autowired dependencies primarily **by type**.

For example:

```java
private Computer computer;
```

Spring looks for a bean compatible with:

```java
Computer
```

Suppose:

```java
interface Computer {
    void compile();
}
```

and:

```java
@Component
public class Laptop implements Computer {
}
```

Spring can inject the `Laptop` bean into:

```java
private Computer computer;
```

because:

```text
Laptop IS-A Computer
```

This is useful because the dependent class can depend on the abstraction/interface rather than a specific implementation.

---

# 14. What If There Are Multiple Implementations?

Suppose we have:

```java
interface Computer {
}
```

and two implementations:

```java
@Component
public class Laptop implements Computer {
}
```

```java
@Component
public class Desktop implements Computer {
}
```

Now:

```java
private Computer computer;
```

has two possible beans:

```text
Computer
   ├── Laptop
   └── Desktop
```

Spring cannot determine which one should be injected automatically.

This results in a **multiple-candidate / ambiguity error**.

---

# 15. @Primary

We can tell Spring which implementation should be preferred:

```java
@Component
@Primary
public class Laptop implements Computer {
}
```

Now, when Spring needs a `Computer` and multiple candidates exist, the `Laptop` bean is preferred.

---

# 16. @Qualifier

Another option is to explicitly specify which bean we want.

For example:

```java
@Component
public class Laptop implements Computer {
}
```

```java
@Component
public class Desktop implements Computer {
}
```

Then:

```java
@Autowired
@Qualifier("laptop")
private Computer computer;
```

`@Qualifier` tells Spring exactly which bean should be injected.

By default, the component name is usually the class name with the first letter converted to lowercase:

```text
Laptop → laptop
Desktop → desktop
```

You can also explicitly specify the bean name:

```java
@Component("myLaptop")
public class Laptop implements Computer {
}
```

Then:

```java
@Qualifier("myLaptop")
```

can be used.

---

# 17. Spring vs Spring Boot

## Spring Framework

Spring is the underlying framework that provides capabilities such as:

- IoC
- Dependency Injection
- Bean management
- Web application support
- Data access
- Transaction management
- Security integration
- etc.

Historically, configuring Spring applications could involve substantial configuration.

For a web application, you could deploy a Spring application to an externally managed servlet container such as Tomcat.

---

## Spring Boot

Spring Boot is built on top of Spring and makes Spring application development easier by providing:

- Auto-configuration
- Starter dependencies
- Embedded servers
- Convention over configuration
- Easier application setup
- Production-oriented features such as Actuator

The important idea is:

> **Spring provides the framework; Spring Boot simplifies configuration and application setup around Spring.**

---

# 18. How Does Spring Boot Run Without Installing Tomcat?

This is an important point.

Spring Boot can package a web application with an **embedded servlet container**.

For example, with the appropriate Spring Boot web dependencies, Tomcat can be included as a dependency inside the application.

So instead of:

```text
Your application
       ↓
Deploy WAR
       ↓
External Tomcat
       ↓
Run application
```

Spring Boot can do:

```text
Spring Boot application
       +
Embedded Tomcat
       ↓
Run application
       ↓
Tomcat starts inside the application
       ↓
HTTP requests are handled
```

So when you run:

```bash
java -jar application.jar
```

the embedded server starts as part of the application.

**Spring Boot does not eliminate Tomcat.**

Rather, for the standard servlet setup, it can **embed Tomcat inside your application**, so you don't have to separately install and manage an external Tomcat server.

---

# 19. How Does It Work During Deployment?

For a typical Spring Boot web application:

```text
application.jar
   │
   ├── Your application classes
   ├── Spring Framework
   ├── Spring Boot
   └── Embedded Tomcat
```

When the application starts:

```text
java -jar application.jar
          ↓
Spring Boot starts
          ↓
Spring ApplicationContext starts
          ↓
Embedded Tomcat starts
          ↓
Spring MVC / controllers are registered
          ↓
Application accepts HTTP requests
```

For example:

```text
GET /users
      ↓
Embedded Tomcat
      ↓
Spring MVC
      ↓
@Controller
      ↓
Service
      ↓
Repository
      ↓
Database
```

---

# 20. Does Spring Create All 1,000 Objects?

**No.**

If you have 1,000 Java classes, Spring doesn't simply create 1,000 objects because the classes exist.

Only classes registered as Spring beans are managed.

For example:

```java
@Component
class Dev {
}
```

```java
@Component
class Laptop {
}
```

Spring manages these.

A normal class:

```java
class Calculator {
}
```

doesn't automatically become a Spring bean merely because it exists in the project.

You can still create it yourself:

```java
Calculator calculator = new Calculator();
```

but Spring won't manage that object as a bean.

---

# 21. Do We Need Configuration for Every Bean?

In traditional Spring, beans could be explicitly configured.

For example, configuration could tell Spring which classes should become beans.

Spring Boot makes this much easier through:

```java
@Component
```

and related stereotype annotations such as:

```java
@Service
@Repository
@Controller
```

Spring Boot's component scanning can discover these classes automatically when they are in the appropriate package structure.

So instead of manually configuring every class, we can mark the classes that Spring should manage.

---

# 22. Java Question — Where Do We Create Objects?

In normal Java, an object can be created anywhere you have executable code, but typically object creation happens inside methods or constructors, depending on the design.

For example:

```java
class Dev {

    private Laptop laptop;

    public Dev() {
        laptop = new Laptop();
    }
}
```

Here the object is created in the constructor.

Or:

```java
public void start() {
    Laptop laptop = new Laptop();
}
```

Here it is created inside a method.

Getters and setters normally **do not create objects**.

A getter usually returns a value:

```java
public Laptop getLaptop() {
    return laptop;
}
```

A setter usually assigns a value:

```java
public void setLaptop(Laptop laptop) {
    this.laptop = laptop;
}
```

So don't think of object creation as something that normally happens in getters/setters.

---

# 23. Complete Dependency Flow

A simple Spring example:

```text
             Spring IoC Container
                     │
          ┌──────────┴──────────┐
          ↓                     ↓
       Dev bean             Laptop bean
          │
          │ dependency
          ↓
       Laptop
```

Code:

```java
@Component
public class Laptop {

    public void compile() {
        System.out.println("This is laptop class");
    }
}
```

```java
@Component
public class Dev {

    private Laptop laptop;

    public Dev(Laptop laptop) {
        this.laptop = laptop;
    }

    public void build() {
        laptop.compile();
        System.out.println("Working on spring project");
    }
}
```

Application:

```java
ApplicationContext context =
        SpringApplication.run(MyAppApplication.class, args);

Dev obj = context.getBean(Dev.class);

obj.build();
```

The important sequence is:

```text
SpringApplication.run()
        ↓
ApplicationContext starts
        ↓
Spring discovers @Component classes
        ↓
Creates Laptop bean
        ↓
Creates Dev bean
        ↓
Sees Dev requires Laptop
        ↓
Injects Laptop into Dev
        ↓
getBean(Dev.class)
        ↓
Dev object is returned
        ↓
obj.build()
        ↓
laptop.compile()
```

# Key Points to Remember

```text
IoC
→ Control of object creation/lifecycle is given to Spring.

IoC Container
→ Manages Spring beans and their dependencies.

Bean
→ An object managed by Spring.

Dependency Injection
→ Spring provides the dependencies required by a bean.

@Component
→ Marks a class as a Spring-managed component.

@Autowired
→ Requests dependency injection.

Constructor Injection
→ Dependency is provided through the constructor.
→ Generally preferred.

Setter Injection
→ Dependency is provided through a setter.

Field Injection
→ Dependency is injected directly into a field.
→ Generally not recommended.

@Primary
→ Gives one bean preference when multiple candidates exist.

@Qualifier
→ Explicitly selects a particular bean.

Spring
→ Provides the underlying framework/features such as IoC and DI.

Spring Boot
→ Simplifies Spring application setup through auto-configuration,
  starters, embedded servers, and conventions.

SpringApplication.run()
→ Starts the Spring application and returns an ApplicationContext.

ApplicationContext
→ Provides access to the Spring IoC container and its beans.

Embedded Tomcat
→ Spring Boot can package Tomcat with a web application,
  so a separate Tomcat installation is not required.
```
