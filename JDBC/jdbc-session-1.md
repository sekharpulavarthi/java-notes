````markdown
# Java Day 5 — JDBC Architecture, Driver Abstraction & CRUD Operations

## 1. Why Persistence? Volatile Memory vs Persistent Storage

Java applications execute in memory managed by the **Java Virtual Machine (JVM)**.

```text
+-------------------------------------------------------------+
|                 Volatile Memory (RAM)                       |
|  - Objects, variables, JVM heap memory                      |
|  - High speed, temporary lifecycle                          |
|  - Data is lost when JVM stops or process terminates        |
+-------------------------------------------------------------+
                              │
                    Requires Persistence
                              │
                              ▼
+-------------------------------------------------------------+
|             Non-Volatile Storage (Hard Disk / SSD)          |
|  Option A: File System (java.io.* / java.nio.*)             |
|  Option B: Relational Database Management System (RDBMS)    |
+-------------------------------------------------------------+
```
````

### File System (`java.io`) vs RDBMS

- **Flat Files (`java.io.*`)**: Suitable for logs and simple dumps, but lacks concurrency control, transactional integrity (ACID), indexing, and relational queries.
- **RDBMS (MySQL, PostgreSQL, Oracle)**: Provides robust data integrity, relational queries via SQL, multi-user concurrency, and ACID transactions.

### Why Learn Core JDBC When We Have Spring JDBC & Hibernate?

Higher-level persistence frameworks do not replace JDBC—they build directly on top of it:

```text
[ Hibernate / JPA ]  ──► Generates SQL & manages ORM state
         │
         ▼
[   Spring JDBC   ]  ──► Eliminates boilerplate & exception translation
         │
         ▼
[    Core JDBC    ]  ──► Manages connections, sockets, statements & result sets
         │
         ▼
[ Database Driver ]  ──► Encodes vendor-specific wire protocol over TCP/IP
         │
         ▼
[   RDBMS Engine  ]  ──► MySQL / Oracle / PostgreSQL

```

Higher-level Java persistence technologies such as Spring JDBC and Hibernate/JPA commonly use JDBC underneath when communicating with relational databases.

---

## 2. JDBC Architecture: Abstraction & Polymorphism

JDBC (**Java Database Connectivity**) is an abstraction layer provided by Java SE via standard interfaces in `java.sql` and `javax.sql`.

### The Abstraction Problem

Every database engine uses different internal protocols, network formats, and proprietary features:

- MySQL communicates over port `3306` using its own binary protocol.
- Oracle uses port `1521` with the TNS protocol.
- PostgreSQL uses port `5432` with its frontend/backend wire protocol.

If developers wrote database-specific code directly, changing a database vendor would require rewriting the entire persistence layer.

### The Solution: Interface-Driven Polymorphism

Java defines the **specifications** (interfaces), and database vendors supply the **implementation** inside their driver JARs:

```text
         Java Application (Your Code)
                      │
                      ▼
         [ java.sql Interfaces ]
    (Connection, Statement, ResultSet)
                      │
   ┌──────────────────┼──────────────────┐
   │ (implements)     │ (implements)     │ (implements)
   ▼                  ▼                  ▼
[MySQL Driver JAR] [Oracle Driver JAR] [PostgreSQL Driver JAR]
(com.mysql.cj.jdbc) (oracle.jdbc.driver)   (org.postgresql)
   │                  │                  │
   ▼                  ▼                  ▼
MySQL Server       Oracle Server       PostgreSQL Server

```

Your code only interacts with the `java.sql` interfaces. The vendor driver translates those standard method calls into vendor-specific network packets.

---

## 3. The 7 Standard Steps of a JDBC Application

```text
Step 1: Import Packages & Add Driver Dependency (pom.xml / build.gradle)
Step 2: Load and Register the Driver
Step 3: Establish the Connection (DriverManager.getConnection)
Step 4: Create a Statement (connect.createStatement)
Step 5: Execute the Query (executeUpdate / executeQuery)
Step 6: Process the Result (iterate over ResultSet if SELECT)
Step 7: Close the Resources (Statement and Connection)

```

---

## 4. Step-by-Step Breakdown

### Step 1: Add Driver Dependency & Import Packages

Import standard JDBC interfaces:

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.Statement;
import java.sql.ResultSet;
import java.sql.SQLException;

```

Add the vendor driver to your project (e.g., in `pom.xml`):

```xml
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>8.3.0</version>
</dependency>

```

### Step 2: Load & Register the Driver

```java
Class.forName("com.mysql.cj.jdbc.Driver");

```

- **What happens**: `Class.forName()` dynamically loads the driver class into JVM memory.
- **Under the hood**: The driver class contains a static initialization block:

```java
static {
    DriverManager.registerDriver(new Driver());
}

```

Loading the class automatically registers an instance with `DriverManager`.

- **Exception**: Throws `ClassNotFoundException` (checked) if the driver JAR is missing from the classpath.
- _Note_: Since JDBC 4.0 (Java 6+), the **Java ServiceLoader mechanism** automatically registers drivers found on the classpath via `META-INF/services/java.sql.Driver`. Explicitly calling `Class.forName()` is optional in modern setups, but remains common practice for explicit configuration.

### Step 3: Establish the Connection

```java
String url = "jdbc:mysql://localhost:3306/jdbclearning";
String user = "root";
String password = "MySql123!";

Connection connect = DriverManager.getConnection(url, user, password);

```

- **JDBC URL Anatomy**:
- `jdbc:` → The main protocol.
- `mysql:` → The sub-protocol (identifies the vendor driver).
- `//localhost:3306/` → Host machine and TCP port.
- `jdbclearning` → Database/schema name.

- **Exception**: Throws `SQLException` (checked) if credentials, network, or database names are incorrect.

### Step 4: Create the Statement

```java
Statement statement = connect.createStatement();

```

- Creates a `Statement` object used for dispatching SQL queries to the database.

### Step 5: Execute the Query

Choose the method based on the SQL query type:

| Method               | SQL Types Handled                                          | Return Type | Description                                                      |
| -------------------- | ---------------------------------------------------------- | ----------- | ---------------------------------------------------------------- |
| `executeUpdate(sql)` | DML (`INSERT`, `UPDATE`, `DELETE`), DDL (`CREATE`, `DROP`) | `int`       | Returns the number of affected rows (or `0` for DDL statements). |
| `executeQuery(sql)`  | DQL (`SELECT`)                                             | `ResultSet` | Returns a tabular cursor pointing to matching records.           |

### Step 6: Process the Results

For `SELECT` queries, traverse the returned `ResultSet`:

```java
ResultSet res = statement.executeQuery("SELECT id, sname, sage, scity FROM studentinfo");

while (res.next()) {
    // Column retrieval by 1-based index OR column label name
    int id = res.getInt("id");            // res.getInt(1)
    String name = res.getString("sname"); // res.getString(2)
    int age = res.getInt("sage");         // res.getInt(3)
    String city = res.getString("scity"); // res.getString(4)

    System.out.println(id + " | " + name + " | " + age + " | " + city);
}

```

- The cursor initially points **before the first row**.
- `res.next()` advances the cursor one row forward. It returns `true` if a row exists, and `false` when no more rows remain.
- **JDBC column indexes are 1-based**, not 0-based.

### Step 7: Close the Resources

Database connections and statements consume native sockets and server handles. They must be explicitly closed:

```java
statement.close();
connect.close();

```

---

## 5. Complete CRUD Implementation

### Traditional Approach (Explicit Close)

```java
package com.meet.jdbclearning;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.Statement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class Launch {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/jdbclearning";
        String user = "root";
        String password = "MySql123!";

        Connection connect = null;
        Statement statement = null;

        try {
            // 1. Load Driver
            Class.forName("com.mysql.cj.jdbc.Driver");

            // 2. Open Connection
            connect = DriverManager.getConnection(url, user, password);

            // 3. Create Statement
            statement = connect.createStatement();

            // ==================== C: CREATE (INSERT) ====================
            String insertSql = "INSERT INTO studentinfo(id, sname, sage, scity) VALUES(2, 'Gupta', 29, 'Hyderabad')";
            int insertedRows = statement.executeUpdate(insertSql);
            System.out.println(insertedRows > 0 ? "Insertion Successful" : "Unable to insert data");

            // ==================== U: UPDATE ====================
            String updateSql = "UPDATE studentinfo SET sage = 30 WHERE id = 2";
            int updatedRows = statement.executeUpdate(updateSql);
            System.out.println(updatedRows > 0 ? "Updation Successful" : "Unable to update data");

            // ==================== R: READ (SELECT) ====================
            String selectSql = "SELECT id, sname, sage, scity FROM studentinfo";
            ResultSet res = statement.executeQuery(selectSql);
            while (res.next()) {
                System.out.println(res.getInt("id") + " | " + res.getString("sname") + " | " + res.getInt("sage") + " | " + res.getString("scity"));
            }
            res.close();

            // ==================== D: DELETE ====================
            String deleteSql = "DELETE FROM studentinfo WHERE id = 2";
            int deletedRows = statement.executeUpdate(deleteSql);
            System.out.println(deletedRows > 0 ? "Deletion Successful" : "Delete failed");

        } catch (ClassNotFoundException e) {
            System.err.println("Driver class missing: " + e.getMessage());
        } catch (SQLException e) {
            System.err.println("Database error: " + e.getMessage());
        } finally {
            // 7. Cleanup in reverse order of creation
            try {
                if (statement != null) statement.close();
                if (connect != null) connect.close();
            } catch (SQLException e) {
                System.err.println("Error closing resources: " + e.getMessage());
            }
        }
    }
}

```

---

## 6. Best Practice: Try-with-Resources (Java 7+)

`Connection`, `Statement`, and `ResultSet` all implement `java.lang.AutoCloseable`. Using **try-with-resources** closes all database handles automatically—even if an exception occurs—without requiring complex `finally` blocks:

```java
package com.meet.jdbclearning;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.Statement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class LaunchModern {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/jdbclearning";
        String user = "root";
        String password = "MySql123!";

        String query = "SELECT id, sname, sage, scity FROM studentinfo";

        // Resources declared here are automatically closed in reverse order
        try (Connection connect = DriverManager.getConnection(url, user, password);
             Statement statement = connect.createStatement();
             ResultSet res = statement.executeQuery(query)) {

            while (res.next()) {
                System.out.println(res.getInt(1) + " " + res.getString(2));
            }

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}

```

---

# Quick Recall

- **Volatile vs Non-Volatile** → JVM memory clears on shutdown; databases provide persistent ACID storage.
- **JDBC Abstraction** → Java supplies standard interfaces (`java.sql.*`); database vendors implement them in their driver JARs.
- **`Class.forName(...)`** → Loads the driver class to trigger its static registration block.
- **JDBC URL** → `jdbc:<subprotocol>://<host>:<port>/<database>`.
- **`executeUpdate()`** → Runs `INSERT`, `UPDATE`, `DELETE`, `CREATE`, `DROP`; returns row count (`int`).
- **`executeQuery()`** → Runs `SELECT`; returns a `ResultSet` cursor.
- **Cursor Mechanics** → `ResultSet` starts _before_ row 1; advance using `res.next()`. Column indexing starts at **`1`**, not `0`.
- **Resource Management** → Always close `ResultSet`, `Statement`, and `Connection` via try-with-resources to prevent connection pool exhaustion.

```

---

### Feedback on Raw Notes & Mistakes Clarified

1. **Terminology: JDBC Expansion**:
   - In your dictated notes, you mentioned: *"DBC Java Database Connection"*.
   - **Correction**: JDBC stands for **Java Database Connectivity**, not Java Database Connection.

2. **Legacy Driver Package Warning**:
   - In your comments, you had: `DriverManager.registerDriver(new com.mysql.jdbc.Driver());`.
   - **Correction**: The package `com.mysql.jdbc.Driver` is the legacy MySQL driver (MySQL 5.x and older). Modern MySQL (version 8.0+) uses `com.mysql.cj.jdbc.Driver`. Using the older package name throws deprecation warnings or `ClassNotFoundException`.

3. **1-Based vs 0-Based Indexing in `ResultSet`**:
   - In standard Java arrays and collections, indexing begins at `0`.
   - In JDBC `ResultSet` methods (`res.getInt(1)`, `res.getString(2)`), column indexing is **1-based**. Passing `0` throws an `SQLException: Column Index out of range`.

4. **Resource Leaks and Closing Order**:
   - In your raw code, you called `statement.close()` and `connect.close()` at the very bottom of the method.
   - If an exception occurs on `statement.executeUpdate(sql)`, the program exits or jumps out immediately, leaving the database connection open on the database server.
   - Always close JDBC resources inside a `finally` block or use Java 7's **try-with-resources** syntax as shown in Section 6.

5. **`PreparedStatement` vs Plain `Statement`**:
   - While plain `Statement` works well for learning basic CRUD operations, concatenation of dynamic values leads to **SQL Injection** and poor database query caching. In production code and upcoming interview prep, prefer using `PreparedStatement` with parameterized placeholders (`?`).

<FollowUp label="Want to cover PreparedStatement and SQL Injection prevention next?" query="Explain PreparedStatement in JDBC: how it prevents SQL Injection and improves performance with parameterized queries."/>

```
