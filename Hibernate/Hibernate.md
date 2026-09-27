# Hibernate

## 1. What is Hibernate?

- Hibernate is an **ORM (Object-Relational Mapping) framework** for Java.
- ORM maps Java objects/classes to database tables and helps us perform database operations without writing SQL for every basic operation.
- Hibernate generates and executes SQL internally.

Example:

```java
Meet m1 = new Meet();

m1.setAid(101);
m1.setName("Sekhar");
m1.setTech("Java");

session.persist(m1);
```

Hibernate can internally generate an SQL `INSERT` for this operation.

Hibernate is **more than just ORM** because it also provides features such as:

- Session management
- Transaction management
- Entity lifecycle/persistence context
- Dirty checking
- Caching
- Query APIs
- Relationship mapping
- Lazy loading

For now, the main thing to remember is:

> **Hibernate is an ORM framework that manages the interaction between Java objects and relational databases.**

---

# 2. Hibernate and JPA

### JPA

JPA stands for **Java Persistence API**.

Today, the specification is called **Jakarta Persistence**, which is why modern applications use packages such as:

```java
import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.Table;
```

JPA is a **specification**, not an ORM implementation.

### Hibernate

Hibernate is an implementation/framework that supports the JPA specification.

So:

```text
JPA / Jakarta Persistence
        ↓
Specification / standard API
        ↓
Hibernate
        ↓
Actual ORM implementation
        ↓
Database
```

Methods such as:

```java
persist()
merge()
find()
```

are part of the JPA/Jakarta Persistence standard and are also available through Hibernate's `Session` API.

---

# 3. Session

`Session` comes from:

```java
org.hibernate.Session
```

A `Session` represents a working context between the Java application and Hibernate/database.

We use it to perform operations such as:

```java
session.persist()
session.find()
session.merge()
session.remove()
```

A Session is obtained from a `SessionFactory`.

---

# 4. SessionFactory

`SessionFactory` comes from:

```java
org.hibernate.SessionFactory
```

It is responsible for creating Sessions.

```java
SessionFactory factory = config.buildSessionFactory();

Session session = factory.openSession();
```

Important:

- `SessionFactory` is relatively expensive to create.
- It is normally created once and reused by the application.
- `Session` is created when we need a database interaction context.
- Both should eventually be closed.

Basic relationship:

```text
Configuration
      ↓
SessionFactory
      ↓
Session
      ↓
Database operations
```

---

# 5. Hibernate Configuration

Hibernate needs configuration information such as:

- Database URL
- Username
- Password
- Database driver
- Database/schema information
- Hibernate properties

The configuration can be loaded using:

```java
Configuration config = new Configuration();

config.configure();
```

By default, Hibernate looks for:

```text
hibernate.cfg.xml
```

If the file is missing:

```text
Could not locate cfg.xml resource [hibernate.cfg.xml]
```

Without sufficient database configuration, Hibernate may not be able to determine how to connect to the database or which SQL dialect to use.

Example error:

```text
Unable to determine Dialect without JDBC metadata
```

---

# 6. Database Configuration

Hibernate needs to know which database it should communicate with.

For example:

```xml
<property name="jakarta.persistence.jdbc.url">
    jdbc:mysql://localhost:3306/meet
</property>

<property name="jakarta.persistence.jdbc.user">
    root
</property>

<property name="jakarta.persistence.jdbc.password">
    password
</property>
```

The exact configuration depends on the database and Hibernate version.

Hibernate uses JDBC underneath to communicate with relational databases.

---

# 7. Entity

An **Entity** is a Java class that Hibernate maps to a database table.

We use:

```java
@Entity
public class Meet {
}
```

This tells Hibernate:

> "Treat this Java class as a persistent entity."

Example:

```java
@Entity
@Table(name = "meet_data")
public class Meet {
    ...
}
```

Here:

```text
Java class → Meet
Database table → meet_data
```

---

# 8. Primary Key — @Id

Every entity needs an identifier.

```java
@Id
private int aid;
```

`@Id` tells Hibernate that this field represents the entity's primary key.

Example:

```java
@Entity
public class Meet {

    @Id
    private int aid;

    private String name;
    private String tech;
}
```

Database representation:

```text
meet
--------------------------------
aid       name       tech
101       Sekhar     Java
102       Mike       Java
```

---

# 9. Mapping Table and Column Names

We can customize the database table name:

```java
@Table(name = "meet_data")
```

We can also customize column names:

```java
@Column(name = "a_id")
private int aid;

@Column(name = "a_name")
private String name;
```

So:

```java
private int aid;
```

can map to:

```text
a_id
```

in the database.

The **entity name** and **table name** are different concepts.

```text
Entity = Java persistence model
Table  = Database structure
```

---

# 10. Registering the Entity

We can explicitly tell Hibernate which class is an entity:

```java
config.addAnnotatedClass(Meet.class);
```

For example:

```java
Configuration config = new Configuration();

config.addAnnotatedClass(Meet.class);
config.configure();
```

This registers `Meet` with Hibernate.

---

# 11. Creating the SessionFactory

After configuration:

```java
SessionFactory factory = config.buildSessionFactory();
```

This creates the `SessionFactory`.

Then:

```java
Session session = factory.openSession();
```

creates a Session.

---

# 12. Persisting an Object

Example:

```java
Meet m1 = new Meet();

m1.setAid(102);
m1.setName("Mike");
m1.setTech("Java");

session.persist(m1);
```

`persist()` makes a transient entity persistent and schedules it to be inserted into the database.

Hibernate can then generate SQL similar to:

```sql
insert into meet_data (a_id, a_name, tech)
values (102, 'Mike', 'Java');
```

The actual SQL depends on the mapping and database.

---

# 13. Transactions

Database write operations should be performed inside a transaction.

Start a transaction:

```java
Transaction transaction = session.beginTransaction();
```

Perform the operation:

```java
session.persist(m1);
```

Commit:

```java
transaction.commit();
```

Typical flow:

```java
Transaction transaction = session.beginTransaction();

session.persist(m1);

transaction.commit();
```

Transactions are important for operations such as:

- INSERT
- UPDATE
- DELETE

A transaction provides a boundary for the database operation and determines when changes are committed.

---

# 14. Complete Basic Example

```java
Meet m1 = new Meet();

m1.setAid(102);
m1.setName("Mike");
m1.setTech("Java");

Configuration config = new Configuration();

config.addAnnotatedClass(Meet.class);
config.configure();

SessionFactory factory = config.buildSessionFactory();

Session session = factory.openSession();

Transaction transaction = session.beginTransaction();

session.persist(m1);

transaction.commit();

session.close();
factory.close();
```

Basic flow:

```text
Create Java object
       ↓
Configure Hibernate
       ↓
Create SessionFactory
       ↓
Open Session
       ↓
Begin Transaction
       ↓
persist()
       ↓
commit()
       ↓
Close Session
       ↓
Close SessionFactory
```

---

# 15. Viewing Generated SQL

We can enable SQL logging using:

```xml
<property name="hibernate.show_sql">true</property>
```

Hibernate will then print generated SQL statements in the console.

---

# 16. Automatically Creating/Updating Tables

Hibernate provides schema-generation settings.

For example:

```xml
<property name="hibernate.hbm2ddl.auto">create</property>
```

### create

Hibernate creates the database schema based on the entity mappings.

Existing schema objects may be dropped/recreated when the SessionFactory starts.

Therefore:

> Do not use `create` for real production data.

### update

```xml
<property name="hibernate.hbm2ddl.auto">update</property>
```

Hibernate attempts to update the schema to match the entity mappings without simply dropping the existing table.

For learning/development, `update` can be convenient.

For production applications, schema migrations are generally handled using dedicated migration tools rather than relying on Hibernate's automatic schema update.

---

# 17. Duplicate Primary Key

If we insert:

```java
m1.setAid(101);
```

when `101` already exists as the primary key, we can get an error such as:

```text
Duplicate entry '101' for key 'PRIMARY'
```

A primary key must uniquely identify a row.

---

# 18. Retrieving Data

We can retrieve an entity using:

```java
Meet m2 = session.find(Meet.class, 101);
```

This means:

> Find the `Meet` entity whose primary key is `101`.

If the entity doesn't exist, `find()` returns:

```java
null
```

Example:

```java
Meet m2 = session.find(Meet.class, 101);

System.out.println(m2);
```

`find()` is the modern JPA-style API.

In Hibernate 7, older APIs such as `get()` and `byId()` are deprecated in favor of `find()`.

---

# 19. Lazy Loading / getReference()

Lazy loading means Hibernate delays loading the actual entity data until it is needed.

Modern Hibernate provides:

```java
session.getReference(Meet.class, 101);
```

This can return a proxy/reference without immediately accessing the database.

When the entity's data is actually accessed, Hibernate can load it.

This is useful when we only need a reference to an entity.

---

# 20. Updating Data

Hibernate supports `merge()` for copying the state of a detached/transient object into a persistent entity.

Example:

```java
Transaction transaction = session.beginTransaction();

Meet m1 = new Meet();

m1.setAid(101);
m1.setName("Sekhar Updated");
m1.setTech("Spring");

Meet updated = session.merge(m1);

transaction.commit();
```

Important:

`merge()` does not simply mean "update this row."

Conceptually, it:

```text
Given object
     ↓
Copy its state to a managed/persistent entity
     ↓
Hibernate synchronizes the managed entity with DB
```

If the object represents an existing entity, its state can be updated.

If the object is unsaved, Hibernate can create a new persistent copy.

Also, the object passed to `merge()` itself does **not** become managed; `merge()` returns the managed instance.

---

# 21. Dirty Checking

An important Hibernate feature is **dirty checking**.

If an entity is already managed by the Session:

```java
Meet m = session.find(Meet.class, 101);

m.setName("New Name");

transaction.commit();
```

We don't necessarily need to explicitly call:

```java
session.update(m);
```

Hibernate tracks changes to managed entities and can generate an `UPDATE` during flush/commit.

This is called:

> **Dirty checking**

Hibernate's Session documentation describes managed entities as being automatically detected for changes and synchronized with the database.

---

# 22. Session Lifecycle / Closing Resources

`SessionFactory` and `Session` consume resources.

Therefore:

```java
session.close();
factory.close();
```

should be performed when they are no longer needed.

In a real Spring Boot application, we normally don't manually manage these objects in this way. Spring manages much of the lifecycle for us.

---

# 23. Hibernate's Basic Architecture

The overall picture is:

```text
Java Entity
     ↓
Hibernate
     ↓
JPA / Hibernate APIs
     ↓
JDBC
     ↓
Database Driver
     ↓
MySQL
```

For example:

```text
Meet object
     ↓
session.persist(m1)
     ↓
Hibernate
     ↓
Generated SQL
     ↓
JDBC
     ↓
MySQL
```

---

# 24. Important Terms

| Term                      | Meaning                                           |
| ------------------------- | ------------------------------------------------- |
| ORM                       | Mapping objects to relational database data       |
| JPA / Jakarta Persistence | Persistence specification/API                     |
| Hibernate                 | ORM framework and JPA implementation              |
| Entity                    | Java class mapped to database data                |
| `@Entity`                 | Marks a class as an entity                        |
| `@Id`                     | Identifies the primary key                        |
| `@Table`                  | Customizes the table mapping                      |
| `@Column`                 | Customizes the column mapping                     |
| SessionFactory            | Creates/manages Hibernate Sessions                |
| Session                   | Working context for persistence operations        |
| Transaction               | Boundary for database operations                  |
| `persist()`               | Makes a transient entity persistent               |
| `find()`                  | Retrieves an entity by ID                         |
| `getReference()`          | Gets a lazy reference/proxy                       |
| `merge()`                 | Copies object state into a managed entity         |
| Dirty checking            | Automatically detects changes to managed entities |
| `hibernate.show_sql`      | Shows generated SQL                               |
| `hbm2ddl.auto`            | Controls schema generation/update behavior        |

---

# 25. The Flow to Remember

For now, the most important Hibernate flow is:

```text
@Entity
   ↓
Configuration
   ↓
SessionFactory
   ↓
Session
   ↓
Transaction
   ↓
persist / find / merge / remove
   ↓
Hibernate generates SQL
   ↓
JDBC
   ↓
Database
   ↓
commit
```

In Spring Boot, much of this manual setup will be replaced by Spring configuration and Spring Data JPA.

The important thing is to understand what is happening underneath.
