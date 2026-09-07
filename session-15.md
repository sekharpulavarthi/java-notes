# Java — Exception Handling

## 1. Types of Errors

At a high level, programming problems can be classified as:

### Compile-Time Error

An error detected **during compilation**.

Examples:

```java
int x = ;                 // syntax error
```

```java
int x = "hello";          // incompatible types
```

```java
System.out.println(x);    // x not declared
```

The program doesn't compile, so bytecode isn't successfully generated.

---

### Runtime Error

The program compiles successfully but encounters a problem **while executing**.

Examples:

```java
int x = 10 / 0;
```

→ `ArithmeticException`

```java
int[] nums = {1, 2, 3};
System.out.println(nums[5]);
```

→ `ArrayIndexOutOfBoundsException`

```java
String s = null;
System.out.println(s.length());
```

→ `NullPointerException`

```java
Class.forName("UnknownClass");
```

→ `ClassNotFoundException`

Runtime problems that are represented by Java's exception mechanism can often be handled using `try-catch`.

---

### Logical Error

The program compiles and runs, but produces the **wrong result because the logic is incorrect**.

Example:

```java
int a = 10;
int b = 20;

int average = a + b / 2;
```

Expected:

```text
15
```

But the expression evaluates to:

```text
20
```

because of operator precedence.

There is no Java exception here. The programmer's logic is wrong.

> **Important:** Division by zero is generally a runtime exception, not a logical error.

---

# 2. What Is an Exception?

An exception is an event that occurs during program execution that **disrupts the normal flow of the program**.

Example:

```java
int i = 0;
int j = 18 / i;
```

This produces:

```text
ArithmeticException
```

Instead of allowing the program to terminate unexpectedly, we can handle it.

---

# 3. `try-catch`

```java
int i = 0;
int j = 0;

try {
    j = 18 / i;
}
catch (Exception e) {
    System.out.println("Something went wrong");
}

System.out.println(j);
System.out.println("Bye");
```

Flow:

```text
try
 ↓
Exception occurs
 ↓
remaining try statements are skipped
 ↓
matching catch executes
 ↓
program continues after catch
```

The code that may cause an exception is placed inside `try`.

The `catch` block handles the exception.

---

# 4. Specific Exception Handling

We can have multiple `catch` blocks:

```java
try {
    j = 18 / i;
    System.out.println(str.length());
    System.out.println(nums[5]);
}
catch (ArithmeticException e) {
    System.out.println("Cannot divide by zero");
}
catch (ArrayIndexOutOfBoundsException e) {
    System.out.println("Stay in your limit.");
}
catch (Exception e) {
    System.out.println("Something went wrong: " + e);
}
```

Each catch handles a particular exception type.

The general `Exception` catch can handle exceptions that weren't caught by the earlier specific catches.

### Order matters

This is correct:

```text
Specific Exception
       ↓
More general Exception
```

This is incorrect:

```java
catch (Exception e) { }
catch (ArithmeticException e) { } // ❌ unreachable
```

because `Exception` already catches `ArithmeticException`.

---

# 5. Exception Hierarchy

The simplified hierarchy is:

```text
Object
   ↓
Throwable
   ├── Error
   └── Exception
        └── RuntimeException
```

More specifically:

```text
Throwable
│
├── Error
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   └── ...
│
└── Exception
    │
    ├── RuntimeException
    │   ├── ArithmeticException
    │   ├── NullPointerException
    │   └── ArrayIndexOutOfBoundsException
    │
    ├── IOException
    ├── SQLException
    └── ...
```

---

# 6. Error vs Exception

### Error

Usually represents a serious problem related to the JVM or runtime environment.

Examples:

```text
OutOfMemoryError
StackOverflowError
```

Application code generally **should not try to handle `Error`**.

### Exception

Represents conditions that application code may need to handle.

Examples:

```text
ArithmeticException
IOException
SQLException
NullPointerException
```

---

# 7. Checked vs Unchecked Exceptions

## Unchecked Exception

These are exceptions derived from `RuntimeException`.

Examples:

```text
ArithmeticException
NullPointerException
ArrayIndexOutOfBoundsException
```

The compiler does **not force** you to handle them.

You can choose whether to handle them.

---

## Checked Exception

Exceptions other than `RuntimeException` (and its subclasses) are generally checked exceptions.

Examples:

```text
IOException
SQLException
ClassNotFoundException
```

The compiler requires you to either:

```text
handle them using try-catch
```

or:

```text
declare them using throws
```

Example:

```java
public void show() throws ClassNotFoundException {
    Class.forName("Calc");
}
```

---

# 8. `throw`

`throw` is used to **explicitly throw an exception**.

Example:

```java
if (age < 18) {
    throw new ArithmeticException("Age is below 18");
}
```

Conceptually:

```text
throw
  ↓
create/choose an exception
  ↓
immediately signal that exception
```

You can throw Java's built-in exception classes or your own custom exception.

---

# 9. Custom Exception

We can create our own exception class.

```java
class NavinException extends Exception {

    public NavinException(String message) {
        super(message);
    }
}
```

Now:

```java
throw new NavinException("Invalid value");
```

creates and throws our custom exception.

Because `NavinException` extends `Exception`, it is a **checked exception**.

---

# 10. `throws`

`throws` is different from `throw`.

### `throw`

Actually throws an exception:

```java
throw new NavinException("Invalid value");
```

### `throws`

Declares that a method **may throw** an exception:

```java
public void show() throws ClassNotFoundException {
    Class.forName("Calc");
}
```

Think:

```text
throw
→ "I am throwing this exception."

throws
→ "This method may throw this exception;
   caller must handle or further declare it."
```

---

# 11. Handling a Checked Exception

Example:

```java
class A {

    public void show() throws ClassNotFoundException {
        Class.forName("Calc");
    }
}
```

Calling it:

```java
A obj = new A();

try {
    obj.show();
}
catch (ClassNotFoundException e) {
    e.printStackTrace();
}
```

Flow:

```text
A.show()
   ↓
Class.forName()
   ↓
ClassNotFoundException may occur
   ↓
throws tells caller about it
   ↓
main() handles it with try-catch
```

---

# 12. `Class.forName()`

```java
Class.forName("Calc");
```

asks Java to load a class by its name.

If the class cannot be found, it can throw:

```text
ClassNotFoundException
```

This is one reason it is useful for demonstrating checked exceptions.

---

# Quick Recall

```text
Compile-time error
→ detected while compiling

Runtime error
→ problem while program is executing

Logical error
→ program runs but produces incorrect result

Exception
→ disrupts normal execution
→ can often be handled

try
→ code that may throw an exception

catch
→ handles an exception

Throwable
├── Error
└── Exception
    └── RuntimeException

Error
→ serious JVM/system-level problems

Checked Exception
→ compiler requires handle OR throws
→ IOException
→ SQLException
→ ClassNotFoundException

Unchecked Exception
→ RuntimeException and subclasses
→ compiler doesn't force handling

throw
→ explicitly throws an exception

throws
→ declares that a method may throw an exception

Custom Exception
→ create a class extending Exception
```
