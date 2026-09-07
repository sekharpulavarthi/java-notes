# Java Day 1 — Introduction, Execution Flow & Data Types

## 1. Java Program Execution

Java source code goes through this flow:

```text
Java Source Code (.java)
        ↓
   javac Compiler
        ↓
   Bytecode (.class)
        ↓
       JVM
        ↓
Machine-specific execution
```

### Important terms

| Term      | Meaning                                                                         |
| --------- | ------------------------------------------------------------------------------- |
| **JDK**   | Java Development Kit — tools required to develop Java applications              |
| **JRE**   | Java Runtime Environment — runtime components required to run Java applications |
| **JVM**   | Java Virtual Machine — executes Java bytecode                                   |
| **javac** | Java compiler — converts `.java` source code into `.class` bytecode             |
| **java**  | Command used to launch a Java application                                       |

### Example

```bash
javac Hello.java
java Hello
```

Compilation:

```text
Hello.java → Hello.class
```

Execution:

```text
Hello.class → JVM → Program runs
```

> `java Hello` uses the **class name**, not `Hello.java` or `Hello.class`.

---

## 2. Platform Independence

Java is called **platform independent** because the same Java bytecode can run on different operating systems.

```text
             Java Source
                  ↓
               javac
                  ↓
              Bytecode
                  ↓
        ┌─────────┼─────────┐
        ↓         ↓         ↓
      JVM       JVM       JVM
    Windows     Linux     macOS
```

### Remember

- **Java bytecode → platform independent**
- **JVM → platform dependent**

Each operating system needs a JVM implementation suitable for that OS.

### WORA

**WORA = Write Once, Run Anywhere**

The same compiled bytecode can run on any supported platform that provides a compatible Java runtime.

---

## 3. JDK, JRE and JVM

Think of the relationship approximately as:

```text
JDK
 └── Runtime environment
      └── JVM
```

### JDK

Used when **developing** Java applications.

It provides tools such as:

- `javac`
- `java`
- `jshell`
- other development/runtime tools

### JRE

Provides the environment required to **run** Java applications.

### JVM

Actually executes the Java bytecode.

> **JDK → Development**
>
> **JRE → Runtime**
>
> **JVM → Executes bytecode**

### Important modern Java note

For modern Java distributions, you should **not think of JRE as a separately installed product in the old Java 8 sense**. Modern JDK distributions generally provide the runtime needed to run Java applications, and standalone JRE distributions are no longer the normal model.

So your original statement:

> "Install JDK → JRE comes with it → JRE contains JVM"

is a useful historical learning model, but don't depend on it too literally for modern Java.

---

## 4. Does Every OS Already Have a JVM?

**No.**

Windows, Linux and macOS do **not automatically mean that a JVM is installed**.

Java/JDK must be installed separately.

Once Java is installed, you can check it using:

```bash
java -version
javac -version
```

---

## 5. Main Method

A Java application commonly starts execution from:

```java
public static void main(String[] args) {
    // code
}
```

### Components

- `public` → accessible to the JVM
- `static` → can be invoked without creating an object
- `void` → doesn't return a value
- `main` → special method name recognized as the program entry point
- `String[] args` → command-line arguments

These are equivalent parameter declarations:

```java
String[] args
String args[]
```

Prefer:

```java
String[] args
```

---

## 6. Multiple Java Files

A project can contain many `.java` files/classes.

The JVM doesn't require the entire project to be contained in one file.

The application has an **entry point**, commonly the `main()` method, from which execution begins.

Example:

```text
Project
 ├── Main.java
 ├── User.java
 ├── Account.java
 └── Transaction.java
```

`Main` may contain:

```java
public static void main(String[] args) {
    // application starts here
}
```

---

# 7. `System.out.print()`

Java provides functionality through classes and methods from its standard library.

For example:

```java
System.out.print("Hello");
System.out.println("Hello");
```

Think of it as:

```text
System
  ↓
out
  ↓
print()
```

`print()` and `println()` are methods used for console output.

---

# 8. JShell

**JShell = Java Shell**

It allows you to interactively execute Java expressions/statements without creating a complete `.java` file and class structure.

Example:

```text
jshell
```

Then:

```java
jshell> 10 + 20
$1 ==> 30

jshell> System.out.println("Hello")
Hello
```

### When is JShell useful?

Good for:

- quickly testing Java syntax
- experimenting with expressions
- learning Java
- trying small pieces of code

It is **not a replacement for normal Java project development**.

### Is JShell available on macOS?

Yes. It is included with standard JDK distributions.

Check:

```bash
jshell
```

Exit:

```text
/exit
```

---

# 9. Java Is Strongly / Statically Typed

Java is a **statically typed, strongly typed language**.

The type of a variable is explicitly declared and checked by the compiler.

Example:

```java
int age = 25;
double price = 99.5;
boolean isActive = true;
char grade = 'A';
```

The general structure is:

```java
type variableName = value;
```

Example:

```java
int age = 25;
```

---

# 10. Statements and Semicolons

A semicolon `;` generally terminates a Java statement.

```java
int age = 25;
System.out.println(age);
```

Missing the required semicolon can result in a compilation error.

> Don't memorize this as "every line needs a semicolon."
> A semicolon terminates statements; not every Java construct ends with one.

---

# 11. Primitive Data Types

Java has **8 primitive data types**:

| Type      |                         Size | Example             |
| --------- | ---------------------------: | ------------------- |
| `byte`    |                       1 byte | `byte b = 10;`      |
| `short`   |                      2 bytes | `short s = 100;`    |
| `int`     |                      4 bytes | `int n = 1000;`     |
| `long`    |                      8 bytes | `long n = 1000L;`   |
| `float`   |                      4 bytes | `float n = 10.5f;`  |
| `double`  |                      8 bytes | `double n = 10.5;`  |
| `char`    |                      2 bytes | `char c = 'A';`     |
| `boolean` | JVM-dependent representation | `boolean b = true;` |

### Easy grouping

```text
Primitive Types
│
├── Integer
│   ├── byte
│   ├── short
│   ├── int
│   └── long
│
├── Floating-point
│   ├── float
│   └── double
│
├── Character
│   └── char
│
└── Boolean
    └── boolean
```

---

# 12. Integer Ranges

### byte

```text
1 byte = 8 bits
Range = -2⁷ to 2⁷ - 1
      = -128 to 127
```

### short

```text
2 bytes = 16 bits
Range = -2¹⁵ to 2¹⁵ - 1
```

### int

```text
4 bytes = 32 bits
Range = -2³¹ to 2³¹ - 1
      = -2,147,483,648 to 2,147,483,647
```

### long

```text
8 bytes = 64 bits
Range = -2⁶³ to 2⁶³ - 1
```

---

# 13. `int` vs `long`

Integer literals are `int` by default.

```java
int x = 100;
```

For a `long` literal, use `L`:

```java
long x = 10000000000L;
```

Prefer uppercase `L` because lowercase `l` can look like `1`.

---

# 14. `float` vs `double`

Floating-point literals are `double` by default.

```java
double price = 10.5;
```

For a `float`, explicitly use `f` or `F`:

```java
float price = 10.5f;
```

---

# 15. `char` vs `String`

`char` represents a **single character**:

```java
char grade = 'A';
```

Character literals use **single quotes**.

```java
'A'
```

A `String` represents a sequence of characters:

```java
String name = "Sekhar";
```

Strings use **double quotes**.

```text
'A'       → char
"Hello"   → String
```

---

# 16. boolean

Java has two boolean values:

```java
true
false
```

Java does **not** treat `0` and `1` as boolean values.

```java
boolean active = true;   // valid
boolean active = false;  // valid
```

---

# 17. Quick Recall

Before moving to the next topic, I should be able to answer:

1. What is the difference between JDK, JRE and JVM?
2. What does `javac` do?
3. What is bytecode?
4. Why is Java platform independent?
5. Why is JVM platform dependent?
6. What does WORA mean?
7. What is the purpose of the `main()` method?
8. What does JShell do?
9. What are Java's 8 primitive data types?
10. What is the difference between `float` and `double`?
11. Why do we use `L` with some `long` literals?
12. Why do we use `f` with `float` literals?
13. What is the difference between `'A'` and `"A"`?
14. Does Java use `0`/`1` for boolean values?
15. What happens when `javac Hello.java` is executed?

---

## One-line Memory Map

```text
.java
  ↓ javac
.class (bytecode)
  ↓ JVM
OS-specific execution

JDK → development
JRE → runtime
JVM → executes bytecode

Java → platform independent
JVM  → platform dependent
WORA → Write Once, Run Anywhere

Primitives:
byte → short → int → long
float → double
char
boolean
```
