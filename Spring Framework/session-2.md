# Spring Framework — XML Configuration, IoC Container & Dependency Injection

## 1. Maven Archetype

A **Maven archetype** is a project template used to generate the initial structure of a Maven project.

It can create the basic folders and files required for a Maven project, for example:

```text
Project
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   └── resources
    └── test
        └── java
```

The archetype provides the initial project structure. We can modify the structure later according to the application's requirements.

---

# 2. Spring Project Without Spring Boot

When using Spring Framework without Spring Boot, we have to configure many things ourselves.

For example:

1. Create a Maven project.
2. Add required Spring dependencies to `pom.xml`.
3. Create the Spring configuration.
4. Create the Spring IoC container.
5. Tell Spring which classes should be managed as beans.
6. Configure dependencies between beans.
7. Obtain beans from the container using `ApplicationContext`.

Spring Boot automates/simplifies many of these steps.

---

# 3. Spring Context Dependency

For XML-based Spring configuration, we need the Spring Context dependency.

The `spring-context` module provides important Spring functionality such as:

- `ApplicationContext`
- `ClassPathXmlApplicationContext`
- Bean management
- Dependency Injection
- Component/configuration infrastructure

The dependency is added to `pom.xml`.

---

# 4. IoC Container

**IoC = Inversion of Control**

In normal Java code, we can create objects ourselves:

```java
Laptop laptop = new Laptop();
Dev dev = new Dev(laptop);
```

The application code controls object creation.

With Spring, this responsibility can be transferred to Spring.

Spring creates and manages objects called **Spring Beans**.

The Spring IoC container is responsible for:

- Creating beans
- Managing beans
- Resolving dependencies
- Injecting dependencies
- Managing bean lifecycle

The Spring IoC container runs inside the JVM.

```text
JVM
 └── Spring Application
      └── Spring IoC Container
           ├── Dev bean
           ├── Laptop bean
           └── Other beans
```

The JVM itself does not provide the Spring IoC container.

---

# 5. ApplicationContext

`ApplicationContext` is an interface provided by Spring.

It represents the Spring application context and provides access to the Spring IoC container and its beans.

Since it is an interface, we don't create it directly:

```java
ApplicationContext context = new ApplicationContext(); // Not possible
```

Instead, we use one of its implementations.

For XML configuration loaded from the classpath:

```java
ApplicationContext context =
        new ClassPathXmlApplicationContext("spring.xml");
```

`ClassPathXmlApplicationContext` is an implementation of `ApplicationContext`.

---

# 6. ClassPathXmlApplicationContext

`ClassPathXmlApplicationContext` loads Spring configuration from an XML file available on the classpath.

Example:

```java
ApplicationContext context =
        new ClassPathXmlApplicationContext("spring.xml");
```

Spring searches the classpath for:

```text
spring.xml
```

For a Maven project, the standard location is:

```text
src/main/resources/spring.xml
```

Maven places resources from this directory on the application's classpath.

---

# 7. Spring XML Configuration

If we want Spring to create and manage objects, we need to tell Spring which classes it should manage.

With XML configuration, we provide this information in a Spring XML file.

Example:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="dev" class="com.meetpeople.Dev">
    </bean>

</beans>
```

The XML configuration tells Spring:

> Create and manage a bean of type `com.meetpeople.Dev` with the bean ID `dev`.

---

# 8. `<beans>` Element

The root element of a Spring bean configuration file is:

```xml
<beans>
```

The Spring XML namespace is specified so that Spring and XML tools know what the Spring-specific elements mean.

```xml
xmlns="http://www.springframework.org/schema/beans"
```

The schema location provides the XML schema definition:

```xml
xsi:schemaLocation="
    http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd"
```

XML itself allows custom tags, but Spring only understands XML elements defined by its configuration vocabulary.

---

# 9. `<bean>` Element

A Spring bean can be declared using:

```xml
<bean id="dev" class="com.meetpeople.Dev">
</bean>
```

Two important attributes are:

### `id`

The name used to identify the bean inside the Spring container.

```xml
id="dev"
```

### `class`

The fully qualified class name.

```xml
class="com.meetpeople.Dev"
```

The fully qualified class name contains:

```text
package name + class name
```

Example:

```text
com.meetpeople.Dev
```

---

# 10. Bean Creation

When Spring reads:

```xml
<bean id="dev" class="com.meetpeople.Dev"/>
```

Spring knows that it should create and manage an object of:

```java
com.meetpeople.Dev
```

Conceptually:

```text
spring.xml
    ↓
Spring reads <bean>
    ↓
Finds com.meetpeople.Dev
    ↓
Creates Dev object
    ↓
Registers/manages it as bean "dev"
```

---

# 11. Getting a Bean Using Its ID

Suppose the XML contains:

```xml
<bean id="dev" class="com.meetpeople.Dev"/>
```

We can obtain it using:

```java
Dev obj = (Dev) context.getBean("dev");
```

`getBean(String)` returns an `Object`, so we need to cast it to `Dev`.

Alternatively, we can provide the type:

```java
Dev obj = context.getBean("dev", Dev.class);
```

This avoids the explicit cast.

---

# 12. Getting a Bean Using Its Type

We can also retrieve a bean using its class:

```java
Dev obj = context.getBean(Dev.class);
```

Spring searches the container for a bean matching the requested type.

This works straightforwardly when there is a single matching bean.

If multiple beans of the same type exist, Spring may not know which one to return and can throw an ambiguity-related exception.

---

# 13. Complete Basic Example

### `Dev.java`

```java
package com.meetpeople;

public class Dev {

    public void build() {
        System.out.println("Working on Spring project");
    }
}
```

### `spring.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="dev" class="com.meetpeople.Dev"/>

</beans>
```

### `Main.java`

```java
package com.meetpeople;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Main {

    public static void main(String[] args) {

        ApplicationContext context =
                new ClassPathXmlApplicationContext("spring.xml");

        Dev obj = context.getBean(Dev.class);

        obj.build();
    }
}
```

Internal flow:

```text
Main
 ↓
ClassPathXmlApplicationContext
 ↓
Load spring.xml from classpath
 ↓
Read bean definitions
 ↓
Create Dev bean
 ↓
Store/manage Dev bean in IoC container
 ↓
context.getBean(Dev.class)
 ↓
Return Dev bean
 ↓
obj.build()
```

---

# 14. What Happens If `spring.xml` Is Missing?

If we write:

```java
new ClassPathXmlApplicationContext("spring.xml");
```

but `spring.xml` is not available on the classpath, Spring cannot load the configuration.

This results in a configuration/resource loading error, commonly involving:

```text
FileNotFoundException
```

or a Spring exception wrapping the resource-not-found problem.

Therefore, the file must be available on the classpath.

Standard Maven location:

```text
src/main/resources/spring.xml
```

---

# 15. What Happens If `spring.xml` Is Empty?

If the XML file is completely empty, it isn't a complete XML document.

Spring/XML parsing can fail with an error such as:

```text
Premature end of file
```

This occurs because the XML parser reaches the end of the file before finding a valid XML document.

---

# 16. Spring Bean

A **Spring Bean** is an object that is instantiated, assembled, and managed by the Spring IoC container.

For example:

```xml
<bean id="dev" class="com.meetpeople.Dev"/>
```

The `Dev` object created and managed by Spring is a Spring Bean.

Not every Java object is automatically a Spring Bean.

An object created manually using:

```java
Dev dev = new Dev();
```

is not automatically managed by Spring.

---

# 17. Multiple Beans of the Same Class

We can define multiple beans using the same class:

```xml
<bean id="dev1" class="com.meetpeople.Dev"/>
<bean id="dev2" class="com.meetpeople.Dev"/>
```

This creates two bean definitions representing two separate bean instances by default:

```text
dev1 → Dev object #1

dev2 → Dev object #2
```

The class is the same, but the bean definitions have different IDs.

---

# 18. Setter Injection

Spring can inject values into an object's properties using setter methods.

Example Java class:

```java
public class Dev {

    private int age;

    public void setAge(int age) {
        this.age = age;
    }

    public int getAge() {
        return age;
    }
}
```

XML:

```xml
<bean id="dev" class="com.meetpeople.Dev">
    <property name="age" value="12"/>
</bean>
```

Spring uses the `setAge()` method to inject the value.

Conceptually:

```text
<property name="age" value="12"/>
             ↓
Spring finds setAge()
             ↓
setAge(12)
             ↓
Dev.age = 12
```

This is called **setter injection**.

---

# 19. `value` in XML

`value` is used when injecting a value such as:

```xml
<property name="age" value="12"/>
```

For example:

```text
int
String
boolean
double
etc.
```

Spring converts the configured value to the required property type when appropriate.

---

# 20. Constructor Injection

Spring can also inject dependencies/values through a constructor.

Example:

```java
public class Dev {

    private int age;

    public Dev(int age) {
        this.age = age;
    }
}
```

XML:

```xml
<bean id="dev" class="com.meetpeople.Dev">
    <constructor-arg value="14"/>
</bean>
```

Spring uses the constructor and supplies `14`.

Conceptually:

```text
Spring
 ↓
Find Dev constructor
 ↓
Pass 14
 ↓
new Dev(14)
```

---

# 21. Multiple Constructor Arguments

If a constructor has multiple arguments, we can specify their indexes.

Example:

```java
public Dev(int age, int experience) {
    ...
}
```

XML:

```xml
<bean id="dev" class="com.meetpeople.Dev">
    <constructor-arg index="0" value="14"/>
    <constructor-arg index="1" value="2"/>
</bean>
```

Indexes are zero-based:

```text
index="0" → first constructor argument

index="1" → second constructor argument
```

---

# 22. `value` vs `ref`

There are two important concepts when injecting dependencies.

### `value`

Used to provide a value:

```xml
<property name="age" value="14"/>
```

### `ref`

Used to refer to another Spring bean:

```xml
<property name="laptop" ref="lap1"/>
```

---

# 23. Reference Injection

Suppose we have:

```java
public class Laptop {

    public void compile() {
        System.out.println("Laptop compiling");
    }
}
```

And:

```java
public class Dev {

    private Laptop laptop;

    public void build() {
        System.out.println("This is Dev class");
        laptop.compile();
    }

    public Laptop getLaptop() {
        return laptop;
    }

    public void setLaptop(Laptop laptop) {
        this.laptop = laptop;
    }
}
```

XML:

```xml
<bean id="dev" class="com.meetpeople.Dev">
    <property name="laptop" ref="lap1"/>
</bean>

<bean id="lap1" class="com.meetpeople.Laptop"/>
```

Here:

```text
dev bean
  |
  | laptop property
  ↓
ref="lap1"
  |
  ↓
lap1 bean
  |
  ↓
Laptop object
```

Spring injects the `Laptop` bean into the `Dev` object's `laptop` property.

---

# 24. How `ref` Wiring Works Internally

Initially, if we manually create:

```java
Dev dev = new Dev();
```

then:

```java
dev.laptop
```

is `null` unless we initialize it.

But with Spring configuration:

```xml
<property name="laptop" ref="lap1"/>
```

Spring performs the wiring while creating/configuring the `Dev` bean.

The process is conceptually:

```text
1. Spring reads the Dev bean definition.

2. Spring sees:
   property name="laptop"

3. Spring sees:
   ref="lap1"

4. Spring searches the container for
   the bean with ID "lap1".

5. Spring obtains the Laptop bean.

6. Spring calls:
   dev.setLaptop(laptopBean)

7. Dev.laptop now refers to the Laptop bean.

8. Later:
   dev.build()

9. laptop.compile() works because
   laptop is no longer null.
```

The JVM executes the Java code, but **Spring performs the dependency wiring**.

---

# 25. Constructor Injection of Another Bean

Dependencies can also be injected through a constructor.

Java:

```java
public class Dev {

    private Laptop laptop;

    public Dev(Laptop laptop) {
        this.laptop = laptop;
        System.out.println("Dev Param constructor");
    }
}
```

XML:

```xml
<bean id="dev" class="com.meetpeople.Dev">
    <constructor-arg ref="lap1"/>
</bean>

<bean id="lap1" class="com.meetpeople.Laptop"/>
```

Conceptually:

```text
lap1 bean
   ↓
Laptop object
   ↓
Dev constructor
   ↓
new Dev(laptop)
```

Spring supplies the `Laptop` dependency while creating the `Dev` bean.

---

# 26. Setter Injection vs Constructor Injection

### Setter Injection

```xml
<property name="laptop" ref="lap1"/>
```

Spring creates/configures the object and then calls the setter.

Conceptually:

```java
Dev dev = new Dev();
dev.setLaptop(laptop);
```

### Constructor Injection

```xml
<constructor-arg ref="lap1"/>
```

Spring provides the dependency while creating the object.

Conceptually:

```java
Dev dev = new Dev(laptop);
```

Constructor injection is useful when the dependency is required for the object to function correctly.

---

# 27. Autowiring

Spring can automatically resolve dependencies without explicitly specifying every dependency using `property` or `constructor-arg`.

In XML configuration, autowiring can be configured using:

```xml
autowire="byName"
```

or:

```xml
autowire="byType"
```

---

# 28. Autowiring by Name

With:

```xml
<bean id="dev"
      class="com.meetpeople.Dev"
      autowire="byName"/>
```

Spring tries to match a bean's name with the property name.

Suppose `Dev` has:

```java
private Laptop laptop;

public void setLaptop(Laptop laptop) {
    this.laptop = laptop;
}
```

Then a bean named `laptop` can be matched:

```xml
<bean id="laptop" class="com.meetpeople.Laptop"/>
```

Conceptually:

```text
Dev property name
      ↓
   laptop
      ↓
Search bean named "laptop"
      ↓
Laptop bean found
      ↓
Inject into Dev.laptop
```

---

# 29. Autowiring by Type

With:

```xml
<bean id="dev"
      class="com.meetpeople.Dev"
      autowire="byType"/>
```

Spring looks at the property type.

If:

```java
private Laptop laptop;
```

and there is exactly one `Laptop` bean, Spring can inject that bean even if its ID is different:

```xml
<bean id="com" class="com.meetpeople.Laptop"/>
```

The ID does not need to match because `byType` uses the type.

```text
byName
    ↓
Bean name ↔ property name

byType
    ↓
Bean type ↔ property type
```

---

# 30. Multiple Beans and Autowiring Ambiguity

Suppose:

```xml
<bean id="laptop" class="com.meetpeople.Laptop"/>
<bean id="laptop2" class="com.meetpeople.Laptop"/>
```

and `Dev` has:

```java
private Laptop laptop;
```

With `autowire="byType"`, Spring finds two beans of type `Laptop`.

Spring cannot determine which one should be injected.

This creates an ambiguity/multiple-candidate error.

---

# 31. `@Primary` and `@Qualifier`

When using annotation-based configuration, Spring provides mechanisms such as:

### `@Primary`

Marks one bean as the preferred candidate when multiple beans match.

### `@Qualifier`

Explicitly identifies which bean should be injected.

These are commonly used with annotation-based dependency injection.

---

# 32. Overall Internal Flow

A Spring application using XML configuration can be understood as:

```text
Maven Project
     ↓
Add Spring dependencies
     ↓
Create spring.xml
     ↓
Define beans
     ↓
Create ClassPathXmlApplicationContext
     ↓
Spring loads spring.xml
     ↓
Spring creates the configured beans
     ↓
Spring resolves dependencies
     ↓
Spring performs dependency injection
     ↓
Beans are managed by the IoC container
     ↓
context.getBean(...)
     ↓
Application receives the required bean
     ↓
Use the bean
```

---

# 33. Important Error Scenarios

### `spring.xml` not found

Cause:

```text
spring.xml
```

is not available on the classpath.

Typical solution:

```text
src/main/resources/spring.xml
```

and load it with:

```java
new ClassPathXmlApplicationContext("spring.xml");
```

---

### `Premature end of file`

Cause:

The XML file is empty or incomplete.

The XML parser reaches the end before finding a complete XML document.

---

### `Cannot find declaration of element 'beans'`

Cause:

The XML parser/IDE does not have the correct Spring XML namespace/schema information.

Ensure the Spring XML configuration contains the correct namespace and schema declaration:

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">
```

---

### Bean not found

If we call:

```java
context.getBean("dev");
```

but no bean with ID `dev` is configured, Spring cannot return that bean.

Check:

```xml
<bean id="dev" class="com.meetpeople.Dev"/>
```

---

### Multiple matching beans

If we request:

```java
context.getBean(Dev.class);
```

and multiple `Dev` beans exist, Spring may not know which one to return.

Similarly, autowiring by type can fail when multiple beans match the required type.

---

# 34. Key Terms

| Term                           | Meaning                                                                           |
| ------------------------------ | --------------------------------------------------------------------------------- |
| Maven Archetype                | Template used to generate an initial Maven project                                |
| IoC                            | Transfer of object creation/management control to Spring                          |
| IoC Container                  | Spring component that creates and manages Spring beans                            |
| Bean                           | Object managed by Spring                                                          |
| ApplicationContext             | Spring interface representing/accessing the application context                   |
| ClassPathXmlApplicationContext | ApplicationContext implementation that loads XML configuration from the classpath |
| `spring.xml`                   | XML file containing Spring bean configuration                                     |
| `bean`                         | XML element used to define a Spring bean                                          |
| `id`                           | Identifier/name of a bean                                                         |
| `class`                        | Fully qualified class name of the bean                                            |
| `getBean()`                    | Retrieves a bean from the Spring container                                        |
| `property`                     | Used for setter/property injection                                                |
| `constructor-arg`              | Used for constructor injection                                                    |
| `value`                        | Supplies a configured value                                                       |
| `ref`                          | Refers to another Spring bean                                                     |
| `autowire="byName"`            | Resolves dependency using bean/property name                                      |
| `autowire="byType"`            | Resolves dependency using bean/property type                                      |

---

# 35. Core Mental Model

The most important thing to remember:

```text
Without Spring:

Developer
   ↓
new Laptop()
   ↓
new Dev(laptop)
   ↓
Developer manages object creation
```

With Spring:

```text
Developer
   ↓
Spring configuration
   ↓
Spring IoC Container
   ↓
creates Laptop
   ↓
creates Dev
   ↓
injects Laptop into Dev
   ↓
Developer gets Dev using getBean()
```

Spring's main job here is to **manage objects and their dependencies so application classes don't have to manually create and connect everything themselves.**
