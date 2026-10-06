# Spring JDBC

## 1. What is JDBC?

**JDBC (Java Database Connectivity)** is a standard Java API used to communicate with relational databases.

Using plain JDBC, the developer generally has to handle:

```text
1. Get database connection
2. Create Statement / PreparedStatement
3. Execute SQL
4. Process ResultSet
5. Handle exceptions
6. Close ResultSet
7. Close Statement
8. Close Connection
```

Typical JDBC flow:

```text
Java Application
      ↓
JDBC API
      ↓
JDBC Driver
      ↓
Database
```

Example:

```java
Connection con = DriverManager.getConnection(url, username, password);

PreparedStatement ps =
        con.prepareStatement("SELECT * FROM alien");

ResultSet rs = ps.executeQuery();

while (rs.next()) {
    // read data
}

rs.close();
ps.close();
con.close();
```

The JDBC API provides the standard interfaces, while the database-specific JDBC driver provides the implementation.

---

# 2. Problems with Plain JDBC

JDBC is powerful, but a lot of repetitive code is required.

For every database operation, developers commonly have to deal with:

- Obtaining connections
- Creating statements
- Setting parameters
- Executing SQL
- Processing `ResultSet`
- Converting database rows into Java objects
- Exception handling
- Closing resources

This results in a lot of **boilerplate code**.

For example:

```text
Connection
PreparedStatement
ResultSet
try/catch
finally
close()
```

Spring JDBC reduces much of this boilerplate.

---

# 3. What is Spring JDBC?

**Spring JDBC** is a Spring module that simplifies working with JDBC.

The main class used is:

```java
JdbcTemplate
```

Spring JDBC still uses JDBC underneath.

It does **not replace JDBC**.

Instead:

```text
Your Application
      ↓
Spring JDBC / JdbcTemplate
      ↓
JDBC API
      ↓
JDBC Driver
      ↓
Database
```

`JdbcTemplate` handles much of the repetitive JDBC work for us.

---

# 4. JDBC vs Spring JDBC

| Plain JDBC                           | Spring JDBC                                                    |
| ------------------------------------ | -------------------------------------------------------------- |
| Developer manages JDBC boilerplate   | Spring handles much of the boilerplate                         |
| Manually obtain/close resources      | `JdbcTemplate` manages resources                               |
| Manually execute statements          | `JdbcTemplate` provides convenient methods                     |
| Manually process results             | `RowMapper` simplifies row-to-object mapping                   |
| More exception handling code         | Spring translates JDBC exceptions                              |
| Connection management is more manual | Usually works with a configured `DataSource` / connection pool |
| More verbose                         | Less verbose                                                   |
| Direct JDBC programming              | Higher-level abstraction over JDBC                             |

### Important

Spring JDBC is **not a completely different database technology**.

It is a convenient abstraction over JDBC.

---

# 5. Does Spring JDBC Reuse Connections?

The idea needs an important clarification.

Spring JDBC itself is not simply:

> "Instead of creating a new connection, Spring JDBC reuses the same connection."

In a typical Spring Boot application, the application uses a **DataSource**, which commonly provides a **connection pool**.

A connection pool maintains a collection of database connections.

Conceptually:

```text
Application
     ↓
JdbcTemplate
     ↓
DataSource / Connection Pool
     ↓
 ┌───────┬───────┬───────┐
 │ Conn1 │ Conn2 │ Conn3 │
 └───────┴───────┴───────┘
     ↓
   Database
```

When a request needs a connection:

```text
Request
   ↓
Get available connection from pool
   ↓
Execute database operation
   ↓
Return connection to pool
```

The connection is normally **returned to the pool**, rather than physically creating a brand-new database connection for every operation.

This improves performance because establishing a database connection can be relatively expensive.

### Important distinction

```text
Spring JDBC
    → Simplifies JDBC programming

Connection Pool
    → Reuses database connections
```

They are related but **not the same concept**.

---

# 6. JdbcTemplate

`JdbcTemplate` is one of the most important classes in Spring JDBC.

It simplifies common database operations such as:

```java
template.update(...)
template.query(...)
template.queryForObject(...)
```

Example:

```java
String sql = "insert into alien (id, name, tech) values (?, ?, ?)";

template.update(
    sql,
    alien.getId(),
    alien.getName(),
    alien.getTech()
);
```

The `?` placeholders are parameters.

This is preferable to constructing SQL by string concatenation.

---

# 7. `JdbcTemplate.update()`

`update()` is generally used for operations that modify data.

Examples:

```text
INSERT
UPDATE
DELETE
```

Example:

```java
int rows = template.update(
    "insert into alien (id, name, tech) values (?, ?, ?)",
    alien.getId(),
    alien.getName(),
    alien.getTech()
);
```

The return value represents the number of affected rows.

For example:

```text
1
```

means one row was affected.

---

# 8. `JdbcTemplate.query()`

`query()` is commonly used when a SQL statement returns multiple rows.

Example:

```java
return template.query(sql, (rs, rowNum) -> {
    Alien a = new Alien();

    a.setId(rs.getInt(1));
    a.setName(rs.getString(2));
    a.setTech(rs.getString(3));

    return a;
});
```

Conceptually:

```text
Database rows
      ↓
ResultSet
      ↓
RowMapper
      ↓
Alien objects
      ↓
List<Alien>
```

---

# 9. RowMapper

`RowMapper` is used to convert an individual database row into a Java object.

Conceptually:

```text
Database Row
     ↓
 RowMapper
     ↓
Alien Object
```

Example:

```java
RowMapper<Alien> mapper = new RowMapper<Alien>() {

    @Override
    public Alien mapRow(ResultSet rs, int rowNum)
            throws SQLException {

        Alien a = new Alien();

        a.setId(rs.getInt(1));
        a.setName(rs.getString(2));
        a.setTech(rs.getString(3));

        return a;
    }
};
```

Then:

```java
List<Alien> aliens = template.query(sql, mapper);
```

The lambda version is simply a shorter way of writing the same mapping logic:

```java
template.query(sql, (rs, rowNum) -> {

    Alien a = new Alien();

    a.setId(rs.getInt(1));
    a.setName(rs.getString(2));
    a.setTech(rs.getString(3));

    return a;
});
```

### Important

`RowMapper` maps **one row at a time**.

`JdbcTemplate.query()` uses that mapper while processing the returned rows and ultimately gives us a collection of mapped objects.

---

# 10. H2 Database

**H2** is a relational database that is commonly used for development, testing, demonstrations, and learning.

It can run as an **embedded/in-memory database**.

Example:

```text
Spring Boot Application
        ↓
      H2 DB
        ↓
    Application memory
```

With an in-memory database, the data generally exists only while the database/application is running.

This makes H2 convenient because we don't need to install and maintain a separate MySQL/PostgreSQL server just to experiment with JDBC.

---

# 11. Why Use H2 for Learning?

H2 is useful because:

- Easy to configure
- Lightweight
- Fast
- Can run in memory
- Good for testing
- No separate database server is required for a basic setup
- Allows us to practice SQL/JDBC/Spring JDBC

For a production application, we would typically use a persistent database such as:

```text
MySQL
PostgreSQL
Oracle
SQL Server
etc.
```

---

# 12. JDBC API vs H2 Database

These are completely different things.

### JDBC API

JDBC is the **Java programming API** used to communicate with databases.

```text
Java
 ↓
JDBC API
```

### H2

H2 is an actual **relational database**.

```text
Application
     ↓
JDBC
     ↓
H2 Database
```

So:

```text
JDBC = mechanism/API for communicating with database

H2 = database
```

---

# 13. Database Driver

To communicate with a particular database through JDBC, we need the appropriate JDBC driver.

For example:

```text
Java Application
      ↓
JDBC API
      ↓
MySQL JDBC Driver
      ↓
MySQL
```

or:

```text
Java Application
      ↓
JDBC API
      ↓
PostgreSQL JDBC Driver
      ↓
PostgreSQL
```

The driver translates JDBC calls into database-specific communication.

---

# 14. Switching from H2 to MySQL/PostgreSQL

Spring Boot can configure database connectivity through properties.

For example:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=password
```

The actual configuration depends on the database.

We also need the appropriate JDBC driver dependency.

For example:

```text
MySQL
    → MySQL JDBC driver

PostgreSQL
    → PostgreSQL JDBC driver
```

Spring Boot then uses the configured datasource.

---

# 15. `schema.sql`

`schema.sql` can contain SQL statements used to create database structures.

Example:

```sql
CREATE TABLE alien(
    id int primary key,
    name varchar(50),
    tech varchar(20)
);
```

This creates:

```text
alien
 ├── id
 ├── name
 └── tech
```

---

# 16. `data.sql`

`data.sql` can contain SQL statements used to insert initial data.

Example:

```sql
insert into alien (id, name, tech)
values (101, 'Rajesh', 'Blockchain');

insert into alien (id, name, tech)
values (102, 'Mahi', 'Java');

insert into alien (id, name, tech)
values (103, 'Kiran', 'Script');
```

Conceptually:

```text
schema.sql
    ↓
Create tables

data.sql
    ↓
Insert initial data
```

---

# 17. Relational Database Structure

A relational database stores data in tables.

Example:

```text
ALIEN TABLE

+-----+--------+------------+
| id  | name   | tech       |
+-----+--------+------------+
| 101 | Rajesh | Blockchain |
| 102 | Mahi   | Java       |
| 103 | Kiran  | Script     |
+-----+--------+------------+
```

A table contains:

- Rows
- Columns

A row represents one record.

A column represents an attribute of the record.

---

# 18. Java Object vs Database Table

A Java class and a database table are **conceptually related but not the same thing**.

For example:

```java
public class Alien {

    private int id;
    private String name;
    private String tech;
}
```

can represent the data stored in:

```text
alien
----------------
id
name
tech
```

One database row can be represented by one `Alien` object:

```text
Database Row
     ↓
Alien Object
```

For example:

```text
101 | Rajesh | Blockchain
```

becomes:

```java
Alien alien = new Alien();

alien.setId(101);
alien.setName("Rajesh");
alien.setTech("Blockchain");
```

Spring JDBC requires us to explicitly write this mapping, commonly using `RowMapper`.

Hibernate/JPA provides much more automatic object-relational mapping.

---

# 19. JDBC vs Spring JDBC vs Hibernate

These are related technologies, but they are not the same.

|                       | JDBC                   | Spring JDBC                   | Hibernate/JPA                        |
| --------------------- | ---------------------- | ----------------------------- | ------------------------------------ |
| Level                 | Low-level database API | JDBC abstraction              | ORM                                  |
| SQL                   | Write SQL              | Write SQL                     | Often generated by ORM               |
| Boilerplate           | High                   | Lower                         | Much lower for common CRUD           |
| Object mapping        | Manual                 | Usually `RowMapper`           | ORM handles mapping                  |
| Connection management | More manual            | Spring manages infrastructure | ORM/framework manages infrastructure |
| Main abstraction      | JDBC APIs              | `JdbcTemplate`                | Entities/ORM                         |
| Best suited for       | Direct SQL control     | SQL-centric applications      | Object-relational applications       |

### The most important distinction

**Spring JDBC still expects you to write SQL.**

Example:

```java
String sql = "SELECT * FROM alien";
```

Hibernate works differently.

You generally define an entity:

```java
@Entity
public class Alien {
    ...
}
```

and Hibernate handles much of the mapping between:

```text
Java Object
      ↕
Database Table
```

---

# 20. Spring Boot vs Spring JDBC vs Hibernate

These terms can be confusing because they operate at different levels.

### Spring Boot

Spring Boot is a framework for simplifying the creation and configuration of Spring applications.

### Spring JDBC

A Spring module that simplifies JDBC-based database access.

### Hibernate

An ORM framework that maps Java objects to relational database tables.

### JPA

JPA is a Java specification/API for ORM.

Hibernate is one of the major implementations of JPA.

So:

```text
Spring Boot
    ↓
Can use Spring JDBC
    ↓
Can use Hibernate/JPA
    ↓
Can use other technologies
```

Spring Boot itself is **not a replacement for Hibernate or JDBC**.

---

# 21. Repository Layer

A common application structure is:

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
Database
```

### Controller

Handles HTTP requests.

```text
GET /products
POST /products
```

### Service

Contains business logic.

```text
Validate
Calculate
Apply business rules
```

### Repository

Handles data-access operations.

For example:

```java
@Repository
public class AlienRepo {

    // database operations
}
```

With Spring JDBC, the repository commonly uses:

```java
JdbcTemplate
```

---

# 22. `@Repository`

`@Repository` tells Spring that the class is a repository/data-access component.

Example:

```java
@Repository
public class AlienRepo {
}
```

Spring creates and manages this class as a bean.

It also participates in Spring's data-access exception translation mechanism.

---

# 23. Spring Bean

A **Spring bean** is an object that is created and managed by the Spring IoC container.

For example:

```java
@Component
public class Alien {
}
```

Spring can create an `Alien` bean.

Similarly:

```java
@Service
public class ProductService {
}
```

creates a Spring-managed service bean.

And:

```java
@Repository
public class AlienRepo {
}
```

creates a Spring-managed repository bean.

---

# 24. Singleton Scope

By default, Spring beans have **singleton scope**.

This means:

> Spring creates one bean instance for that bean definition within a particular Spring ApplicationContext.

Example:

```java
@Component
public class Alien {
}
```

By default:

```text
Spring Container

Alien bean
    ↓
One Alien object
```

If multiple Spring-managed components request that bean, Spring normally gives them the same bean instance.

### Important

Spring singleton does **not** mean one object for the entire JVM/application universe.

It means one bean instance **per Spring ApplicationContext**.

---

# 25. Prototype Scope

We can specify:

```java
@Scope("prototype")
```

Example:

```java
@Component
@Scope("prototype")
public class Alien {
}
```

Now, whenever the Spring container is asked to create/get that prototype bean, Spring creates a **new instance**.

Conceptually:

```text
Singleton:

getBean(Alien.class)
        ↓
     Alien #1

getBean(Alien.class)
        ↓
     Alien #1
```

Prototype:

```text
getBean(Alien.class)
        ↓
     Alien #1

getBean(Alien.class)
        ↓
     Alien #2

getBean(Alien.class)
        ↓
     Alien #3
```

---

# 26. Why Prototype Can Be Useful Here

Suppose `Alien` contains mutable state:

```java
private int id;
private String name;
private String tech;
```

If the same object is reused for multiple pieces of independent state:

```text
Alien object
   ↓
id = 101
name = Rajesh
```

and later the same object is changed:

```text
id = 102
name = Mahi
```

the original state has been overwritten.

Prototype scope gives a fresh object when the container creates the bean.

### Important clarification

Prototype scope does **not** mean:

> "Spring creates a new object for every HTTP request."

That is not what prototype scope itself means.

It means:

> A new instance is created each time the Spring container is asked to create that prototype bean.

Request scope is a different Spring bean scope and is specifically associated with web requests.

---

# 27. Singleton and Thread Safety

Singleton beans are shared objects.

Therefore, singleton beans should generally **not store request-specific mutable state in instance fields**.

For example, this is dangerous design:

```java
@Service
public class UserService {

    private String currentUser;
}
```

If multiple requests use the same singleton service concurrently, they could interfere with each other's state.

A better approach is generally to keep request-specific data in:

```text
method parameters
local variables
request-scoped objects
```

rather than mutable singleton fields.

---

# 28. `ApplicationContext`

The Spring Application Context is responsible for managing Spring beans and their lifecycle.

Example:

```java
ApplicationContext context =
        SpringApplication.run(
                SpringJdbcApplication.class,
                args
        );
```

Then:

```java
AlienRepo repo = context.getBean(AlienRepo.class);
```

asks the Spring container for the `AlienRepo` bean.

Similarly:

```java
Alien alien = context.getBean(Alien.class);
```

asks Spring for an `Alien` bean.

---

# 29. Your Spring JDBC Application Flow

Your application roughly works like this:

```text
Spring Boot starts
       ↓
ApplicationContext created
       ↓
Spring scans components
       ↓
Creates/manages beans
       ↓
Creates JdbcTemplate
       ↓
Creates AlienRepo
       ↓
Creates Alien prototype bean when requested
       ↓
Application code gets beans from context
       ↓
AlienRepo.save(alien)
       ↓
JdbcTemplate
       ↓
DataSource / Connection Pool
       ↓
JDBC
       ↓
H2
```

---

# 30. Your `save()` Method

```java
public void save(Alien alien) {

    String sql =
        "insert into alien (id, name, tech) values (?, ?, ?)";

    int rows = template.update(
        sql,
        alien.getId(),
        alien.getName(),
        alien.getTech()
    );
}
```

Flow:

```text
Alien object
     ↓
Get values
     ↓
SQL parameters
     ↓
JdbcTemplate.update()
     ↓
JDBC
     ↓
H2
     ↓
INSERT
```

The `?` placeholders are bound to the supplied values.

This also avoids constructing SQL by directly concatenating user-provided values.

---

# 31. Your `findAll()` Method

```java
public List<Alien> findAll() {

    String sql = "SELECT * FROM ALIEN";

    return template.query(sql, (rs, rowNum) -> {

        Alien a = new Alien();

        a.setId(rs.getInt(1));
        a.setName(rs.getString(2));
        a.setTech(rs.getString(3));

        return a;
    });
}
```

Flow:

```text
SELECT query
     ↓
Database
     ↓
ResultSet
     ↓
RowMapper
     ↓
Alien object for each row
     ↓
List<Alien>
```

---

# 32. Why `JdbcTemplate` Is Useful

Without Spring JDBC:

```text
Get Connection
Create PreparedStatement
Set parameters
Execute SQL
Process ResultSet
Handle SQLException
Close ResultSet
Close Statement
Close Connection
```

With Spring JDBC:

```java
template.update(sql, values);
```

or:

```java
template.query(sql, rowMapper);
```

Spring handles much of the repetitive infrastructure code.

You still control the SQL.

This is one of the main reasons to use Spring JDBC.

---

# 33. When Spring JDBC Can Be Preferred

Spring JDBC can be useful when:

- You want direct control over SQL.
- Queries are complex or database-specific.
- You are comfortable writing SQL.
- You don't need full ORM behavior.
- You want less JDBC boilerplate.
- You want Spring's dependency injection and transaction infrastructure.

Hibernate/JPA can be useful when:

- Your application is strongly object-oriented.
- You have many entities and relationships.
- You want ORM mapping.
- You want to reduce repetitive CRUD SQL.
- You want Hibernate to manage much of the object-relational mapping.

Neither technology is universally required. The choice depends on the application's needs.

---

# 34. Core Mental Model

Remember this:

```text
JDBC
↓
Low-level Java database API

Spring JDBC
↓
Simplifies JDBC
↓
JdbcTemplate
↓
Still write SQL

Hibernate/JPA
↓
ORM
↓
Maps Java objects ↔ database tables
```

And:

```text
Spring Boot
↓
Application framework/configuration
↓
Can integrate with JDBC
Can integrate with Spring JDBC
Can integrate with Hibernate/JPA
Can integrate with other technologies
```

---

# 35. Important Interview Questions

### Q1. What is JDBC?

JDBC is a Java API for communicating with relational databases.

### Q2. What is Spring JDBC?

Spring JDBC is a Spring abstraction over JDBC that reduces JDBC boilerplate, primarily through `JdbcTemplate`.

### Q3. Does Spring JDBC replace JDBC?

No. Spring JDBC uses JDBC underneath.

### Q4. What is JdbcTemplate?

`JdbcTemplate` is a Spring class that simplifies database operations such as querying and updating through JDBC.

### Q5. Does Spring JDBC eliminate SQL?

No. Spring JDBC generally requires us to write SQL.

### Q6. What is RowMapper?

`RowMapper` maps a database row from a `ResultSet` into a Java object.

### Q7. What is H2?

H2 is a relational database that can run embedded and in memory, making it convenient for development and testing.

### Q8. What is the difference between JDBC and H2?

JDBC is a Java API for database access. H2 is an actual database.

### Q9. What is a connection pool?

A connection pool maintains reusable database connections so applications don't have to establish a completely new physical database connection for every operation.

### Q10. What is the default scope of a Spring bean?

Singleton.

### Q11. What is prototype scope?

Prototype scope tells Spring to create a new bean instance whenever the container is asked to create that prototype bean.

### Q12. Is prototype scope the same as request scope?

No.

Prototype is based on bean creation requests. Request scope associates a bean instance with an HTTP request.

### Q13. What is the difference between Hibernate and Spring JDBC?

Spring JDBC is a JDBC abstraction where we generally write SQL ourselves. Hibernate is an ORM framework that maps Java objects to relational database tables.

### Q14. What is the difference between JPA and Hibernate?

JPA is a specification/API for ORM. Hibernate is an implementation of JPA.

### Q15. Why do we use a repository layer?

The repository/data-access layer isolates database-access logic from the service and controller layers.
