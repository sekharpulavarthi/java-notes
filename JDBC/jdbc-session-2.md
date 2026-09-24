````markdown
# Java Day 6 — Modular JDBC Design, PreparedStatement & Batch Updates

## 1. Boilerplate Elimination: The `JDBCUtil` Pattern

In production applications, database connection details, driver loading, and cleanup routines should not be duplicated across business methods. Encapsulate connection management inside a dedicated utility class.

### Robust `JDBCUtil` Implementation

```java
package com.meet.jdbclearning;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class JDBCUtil {

    // 1. Static block guarantees driver class loads once per classloader lifecycle
    static {
        try {
            Class.forName("com.mysql.cj.jdbc.Driver");
        } catch (ClassNotFoundException e) {
            throw new ExceptionInInitializerError("MySQL Driver not found on classpath: " + e.getMessage());
        }
    }

    private JDBCUtil() {
        // Prevent direct instantiation of utility class
    }

    // 2. Factory method returning a newly established Connection
    public static Connection getConnection() throws SQLException {
        String url = "jdbc:mysql://localhost:3306/jdbclearning";
        String user = "root";
        String password = "MySql123!";

        return DriverManager.getConnection(url, user, password);
    }

    // 3. Null-safe resource closing with independent try-catch blocks
    public static void closeResource(Connection connect, Statement statement, ResultSet resultSet) {
        if (resultSet != null) {
            try {
                resultSet.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (statement != null) {
            try {
                statement.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (connect != null) {
            try {
                connect.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }

    // Overloaded helper for queries that do not produce a ResultSet (INSERT/UPDATE/DELETE)
    public static void closeResource(Connection connect, Statement statement) {
        closeResource(connect, statement, null);
    }
}
```
````

---

## 2. Dynamic Execution with `statement.execute()`

While `executeQuery()` is strictly for `SELECT` queries and `executeUpdate()` is strictly for DML/DDL, `statement.execute()` is a **generic dispatcher** capable of handling queries of unknown type at runtime.

### How `execute()` Evaluates Output

```text
statement.execute(sql)
       │
       ├── Returns TRUE  ──► Output is a ResultSet (DQL / SELECT)
       │                     Retrieve via: statement.getResultSet()
       │
       └── Returns FALSE ──► Output is an update count or no result (DML / DDL)
                             Retrieve via: statement.getUpdateCount()

```

### Complete Generic Execution Example

```java
package com.meet.jdbclearning;

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class GenericExecutionDemo {
    public static void main(String[] args) {
        Connection connect = null;
        Statement statement = null;
        ResultSet res = null;

        try {
            connect = JDBCUtil.getConnection();
            statement = connect.createStatement();

            String sql = "SELECT id, sname, sage, scity FROM studentinfo";
            boolean isResultSet = statement.execute(sql);

            if (isResultSet) {
                res = statement.getResultSet();
                while (res.next()) {
                    System.out.println(res.getInt("id") + " | "
                                     + res.getString("sname") + " | "
                                     + res.getInt("sage") + " | "
                                     + res.getString("scity"));
                }
            } else {
                int rows = statement.getUpdateCount();
                System.out.println("Rows affected: " + rows);
            }

        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            JDBCUtil.closeResource(connect, statement, res);
        }
    }
}

```

---

## 3. `Statement` vs `PreparedStatement`

The `PreparedStatement` interface extends `Statement` (`java.sql.PreparedStatement extends java.sql.Statement`).

| Architectural Concern     | `Statement`                                              | `PreparedStatement`                                                  |
| ------------------------- | -------------------------------------------------------- | -------------------------------------------------------------------- |
| **Creation**              | `connect.createStatement()` (No SQL passed upfront)      | `connect.prepareStatement(sqlWithPlaceholders)` (SQL passed upfront) |
| **Compilation Phase**     | Compiled **every time** the query executes               | Precompiled by DB engine **once**; parameter values injected later   |
| **Execution Performance** | High overhead for repeated operations                    | Fast for repeated queries (uses DB precompiled execution plan)       |
| **Parameter Handling**    | Manual string concatenation (`' " + val + " '`)          | Parameterized positional placeholders (`?`)                          |
| **Security**              | Vulnerable to **SQL Injection Attacks**                  | Prevents SQL injection by default via strict type escaping           |
| **Lifecycle Reusability** | One `Statement` can execute multiple _different_ queries | Tied to **one SQL blueprint**; only parameters change                |

### Compilation Lifecycle Breakdown

```text
[Statement Workflow]
SQL String with data ──► Sent to DB ──► Parse & Compile ──► Execute Plan ──► Return Data
(Runs entire cycle for EVERY execution, even if only the ID changed)

[PreparedStatement Workflow]
Phase 1 (Preparation):
  SQL Template ("...WHERE id = ?") ──► Sent to DB ──► Parsed, Compiled & Cached Plan Saved
Phase 2 (Execution):
  Parameter values only (e.g., id=2) ──► Sent to DB ──► Reuses Cached Plan ──► Return Data

```

### Why PreparedStatement Stops SQL Injection

When using string concatenation:

```java
// VULNERABLE: If input is "admin' OR '1'='1", authentication is bypassed!
String query = "SELECT * FROM users WHERE user = '" + username + "' AND pass = '" + password + "'";

```

With `PreparedStatement`:

```java
String query = "SELECT * FROM users WHERE user = ? AND pass = ?";
pstmnt = connect.prepareStatement(query);
pstmnt.setString(1, username);
pstmnt.setString(2, password);

```

The database engine treats the input assigned to `?` strictly as **literal data**, never as executable SQL instructions. Special characters (like quotes or semicolons) are escaped automatically.

---

## 4. Full CRUD Operations Using `PreparedStatement`

### Model Class

```java
package com.meet.jdbclearning;

public class Student {
    private int id;
    private String name;
    private int age;
    private String city;

    public Student(int id, String name, int age, String city) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.city = city;
    }

    public int getId() { return id; }
    public String getName() { return name; }
    public int getAge() { return age; }
    public String getCity() { return city; }
}

```

### Complete CRUD Repository Demo

```java
package com.meet.jdbclearning;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class PreparedStatementCRUD {

    // 1. CREATE (INSERT)
    public static void insertStudent(Student s) {
        String sql = "INSERT INTO studentinfo(id, sname, sage, scity) VALUES (?, ?, ?, ?)";
        try (Connection con = JDBCUtil.getConnection();
             PreparedStatement ps = con.prepareStatement(sql)) {

            ps.setInt(1, s.getId());
            ps.setString(2, s.getName());
            ps.setInt(3, s.getAge());
            ps.setString(4, s.getCity());

            int count = ps.executeUpdate();
            System.out.println("Insert status: " + (count > 0 ? "Success" : "Failed"));

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    // 2. READ (SELECT)
    public static void fetchStudentById(int id) {
        String sql = "SELECT id, sname, sage, scity FROM studentinfo WHERE id = ?";
        try (Connection con = JDBCUtil.getConnection();
             PreparedStatement ps = con.prepareStatement(sql)) {

            ps.setInt(1, id);

            try (ResultSet rs = ps.executeQuery()) {
                if (rs.next()) {
                    System.out.println("Student Found -> ID: " + rs.getInt("id")
                            + ", Name: " + rs.getString("sname")
                            + ", Age: " + rs.getInt("sage")
                            + ", City: " + rs.getString("scity"));
                } else {
                    System.out.println("No student found with ID: " + id);
                }
            }

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    // 3. UPDATE
    public static void updateStudentCity(int id, String newCity) {
        String sql = "UPDATE studentinfo SET scity = ? WHERE id = ?";
        try (Connection con = JDBCUtil.getConnection();
             PreparedStatement ps = con.prepareStatement(sql)) {

            ps.setString(1, newCity);
            ps.setInt(2, id);

            int count = ps.executeUpdate();
            System.out.println("Update status: " + (count > 0 ? "Success" : "No record found"));

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    // 4. DELETE
    public static void deleteStudent(int id) {
        String sql = "DELETE FROM studentinfo WHERE id = ?";
        try (Connection con = JDBCUtil.getConnection();
             PreparedStatement ps = con.prepareStatement(sql)) {

            ps.setInt(1, id);

            int count = ps.executeUpdate();
            System.out.println("Delete status: " + (count > 0 ? "Success" : "No record found"));

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        Student newStudent = new Student(101, "Aakash", 24, "Bengaluru");
        insertStudent(newStudent);
        fetchStudentById(101);
        updateStudentCity(101, "Chennai");
        deleteStudent(101);
    }
}

```

---

## 5. High-Performance Batch Processing: `executeBatch()`

### The Network Bottleneck Problem

Executing single insert/update operations inside a loop generates a network round-trip for every single row:

```text
Loop Iteration 1 ──► [Network Call] ──► Database Executed (1 row)
Loop Iteration 2 ──► [Network Call] ──► Database Executed (1 row)
Loop Iteration 3 ──► [Network Call] ──► Database Executed (1 row)
...
(1,000 inserts = 1,000 network round trips -> Severe Performance Degradation)

```

### The Solution: Batch Updates

`addBatch()` bundles multiple parameter sets into a local memory buffer. A single call to `executeBatch()` dispatches the entire bundle across the network at once:

```text
[Client Memory Buffer]
  Row 1 (101, "A") ──► addBatch()
  Row 2 (102, "B") ──► addBatch()
  Row 3 (103, "C") ──► addBatch()
          │
          └─── single executeBatch() ──► [Network Call] ──► Database executes all 3 rows

```

### `executeBatch()` Working Example

```java
package com.meet.jdbclearning;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.util.Arrays;

public class BatchProcessingDemo {
    public static void main(String[] args) {
        String sql = "INSERT INTO studentinfo(id, sname, sage, scity) VALUES (?, ?, ?, ?)";

        Connection connect = null;
        PreparedStatement pstmnt = null;

        try {
            connect = JDBCUtil.getConnection();

            // Disable auto-commit to run batch inside an explicit transaction
            connect.setAutoCommit(false);

            pstmnt = connect.prepareStatement(sql);

            // Record 1
            pstmnt.setInt(1, 201);
            pstmnt.setString(2, "Rahul");
            pstmnt.setInt(3, 23);
            pstmnt.setString(4, "Pune");
            pstmnt.addBatch(); // Enqueue to batch buffer

            // Record 2
            pstmnt.setInt(1, 202);
            pstmnt.setString(2, "Pooja");
            pstmnt.setInt(3, 22);
            pstmnt.setString(4, "Mumbai");
            pstmnt.addBatch(); // Enqueue to batch buffer

            // Record 3
            pstmnt.setInt(1, 203);
            pstmnt.setString(2, "Kiran");
            pstmnt.setInt(3, 25);
            pstmnt.setString(4, "Delhi");
            pstmnt.addBatch(); // Enqueue to batch buffer

            // Dispatch all queued operations over the network in one call
            int[] updateCounts = pstmnt.executeBatch();

            // Explicitly commit the transaction
            connect.commit();

            System.out.println("Batch executed successfully. Rows affected array: " + Arrays.toString(updateCounts));

        } catch (SQLException e) {
            try {
                if (connect != null) {
                    System.err.println("Transaction rolled back due to error.");
                    connect.rollback(); // Rollback if batch fails
                }
            } catch (SQLException ex) {
                ex.printStackTrace();
            }
            e.printStackTrace();
        } finally {
            JDBCUtil.closeResource(connect, pstmnt);
        }
    }
}

```

- **Return value of `executeBatch()**`: Returns an array of integers (`int[]`). Each element contains the row count affected by its corresponding command in the batch.
- **Transaction Safety**: Pair batch execution with `connect.setAutoCommit(false)` and `connect.commit()`. If one record fails midway, `connect.rollback()` ensures the database remains uncorrupted.

---

# Quick Recall

- **`JDBCUtil`** → Centralizes connection acquisition, URL/credentials, and resource cleanup.
- **`statement.execute()`** → Generic method: returns `true` for `ResultSet` (`getResultSet()`), `false` for update count (`getUpdateCount()`).
- **Inheritance** → `PreparedStatement` extends `Statement`.
- **Precompilation** → PreparedStatement query templates are parsed/compiled **once** by the RDBMS engine.
- **SQL Injection Prevention** → Placeholders (`?`) ensure input is treated strictly as value literals, not executable SQL syntax.
- **Positional Indexing** → `pstmnt.setXxx(parameterIndex, value)` is **1-based** (first placeholder is index `1`).
- **`addBatch()`** → Stages dynamic queries in an in-memory batch.
- **`executeBatch()`** → Sends all buffered commands over the network in a single call, returning an `int[]` of affected rows.
- **Batch Transactions** → Always disable `autoCommit(false)`, invoke `commit()`, and catch failures with `rollback()`.

````

---

### Feedback on Raw Notes & Mistakes Clarified

1. **Flaw in `closeConnection` Method**:
   - In your `JDBCUtil.closeConnection(Connection connect, Statement statement)`:
     ```java
     statement.close();
     connect.close();
     ```
   - **The Bug**: If `statement.close()` throws an `SQLException`, the method exits prematurely, and `connect.close()` **never gets called**, leaking a database connection.
   - **Fix**: Each resource must either be closed in its own separate `try-catch` block (as demonstrated in Section 1's `closeResource` method), or handled using Java 7's **try-with-resources**.

2. **ResultSet Leakage in Resource Cleanup**:
   - In `Launch4`, your code processed a `ResultSet` via `ResultSet res = statement.getResultSet();`, but the `finally` block only closed `connect` and `statement`.
   - ResultSets hold server-side cursors and row buffers. Failing to explicitly close the `ResultSet` can cause memory leaks on busy database instances.

3. **Polymorphic Parameter Cleanup (`Statement` vs `PreparedStatement`)**:
   - In `PreparedStatementDemo`, you passed `pstmnt` into `JDBCUtil.closeConnection(connect, pstmnt);`.
   - This works seamlessly because `PreparedStatement` is a child interface of `Statement` (`PreparedStatement extends Statement`). Polymorphism allows `Statement` references to accept `PreparedStatement` instances.

4. **Why PreparedStatement is Useful (Nuance Correction)**:
   - In your notes, you mentioned: *"If we want to read values from any file or anything, then a prepared statement will be useful."*
   - Reading from a file is just one arbitrary data source. The core architectural reasons to use `PreparedStatement` are:
     1. **Security**: Complete immunization against SQL Injection.
     2. **Performance**: DB query cache reuse through precompilation.
     3. **Clean Code**: Elimination of manual quotes and string concatenation bugs.

5. **Batch Processing with AutoCommit**:
   - When using `executeBatch()`, leaving `autoCommit` set to `true` (the default) causes the database to commit each record individually behind the scenes, negating much of the performance advantage. Always wrap batches with `connect.setAutoCommit(false)` and `connect.commit()`.

<FollowUp label="Want to cover Transaction Management (ACID, Commit, Rollback, Savepoints) next?" query="Explain Transaction Management in JDBC: ACID properties, commit, rollback, and savepoints with code examples."/>

````
