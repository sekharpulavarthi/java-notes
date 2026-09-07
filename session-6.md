# Java Day 6 — Strings, StringBuffer, StringBuilder & Encapsulation

## 1. String in Java

`String` is a **class**, not a primitive data type.

```java
String name = "Sekhar";
```

Primitive types:

```java
int
float
double
char
boolean
byte
short
long
```

`String` starts with an uppercase `S` because it is a class provided by Java.

```text
Primitive type → int
Class          → String
```

A `String` variable is a **reference variable** that refers to a String object.

---

# 2. String Constant Pool

Java maintains a special pool called the **String Constant Pool** for String literals.

Example:

```java
String s1 = "Hello";
String s2 = "Hello";
```

The same String literal can be reused:

```text
s1 ──────┐
         ↓
      "Hello"
         ↑
s2 ──────┘
```

Instead of unnecessarily creating another identical String object for the literal `"Hello"`, Java can reuse the existing pooled String.

Therefore:

```java
System.out.println(s1 == s2);
```

produces:

```text
true
```

because both references point to the same pooled String object.

### Important

The String Constant Pool is associated with the JVM's runtime class/data environment and is commonly described as being part of the heap in modern Java.

---

# 3. String Immutability

Strings in Java are **immutable**.

Once a String object is created, its contents cannot be changed.

Example:

```java
String name = "Hello";

name = "World";
```

The original `"Hello"` String object is not modified.

Conceptually:

```text
Before:

name ─────→ "Hello"


After:

name ─────→ "World"

"Hello" remains unchanged.
```

If the old object is no longer reachable, it can eventually be removed by the **Garbage Collector**.

### Remember

```text
String → immutable
```

Operations that appear to modify a String actually produce another String rather than modifying the existing String.

---

# 4. Why Strings Are Immutable

Immutability provides useful properties such as:

- String literals can safely be shared
- Strings can be reused through the String Pool
- String values cannot unexpectedly change through another reference
- Useful for security and thread-safety

---

# 5. StringBuffer

`StringBuffer` is a **mutable sequence of characters**.

Unlike `String`, its contents can be modified.

```java
StringBuffer sb = new StringBuffer("Hello");

sb.append(" World");
```

Now:

```text
Hello World
```

Common operations:

```java
sb.append("Java");
sb.insert(5, "!");
sb.delete(0, 2);
```

---

# 6. StringBuilder

`StringBuilder` is also a **mutable sequence of characters**.

```java
StringBuilder sb = new StringBuilder("Hello");

sb.append(" World");
```

It is commonly preferred when you need to perform many String modifications in a **single-threaded context**.

---

# 7. StringBuffer vs StringBuilder

|                 | StringBuffer                               | StringBuilder        |
| --------------- | ------------------------------------------ | -------------------- |
| Mutable         | Yes                                        | Yes                  |
| Thread-safe     | Yes                                        | No                   |
| Synchronization | Yes                                        | No                   |
| Usually faster  | No                                         | Yes                  |
| Typical use     | Multiple threads sharing the same instance | Single-threaded code |

### What does "thread-safe" mean?

A **thread** is an independent path of execution within a program.

If multiple threads access the same mutable object at the same time, their operations can potentially interfere with each other.

`StringBuffer` synchronizes its relevant methods, making concurrent access safer.

`StringBuilder` does not provide that synchronization, so it generally has less overhead and is faster when thread safety isn't required.

### Memory trick

```text
String        → Immutable
StringBuffer  → Mutable + Thread-safe
StringBuilder → Mutable + Not synchronized
```

---

# 8. Converting StringBuffer/StringBuilder to String

Use `toString()`:

```java
StringBuffer sb = new StringBuffer("Hello");

String str = sb.toString();
```

Similarly:

```java
StringBuilder sb = new StringBuilder("Hello");

String str = sb.toString();
```

You don't need to create two Strings.

---

# 9. StringBuffer Capacity

When creating a `StringBuffer` without specifying capacity:

```java
StringBuffer sb = new StringBuffer();
```

the default initial capacity is **16 characters**.

If you provide an initial String:

```java
StringBuffer sb = new StringBuffer("Hello");
```

its initial capacity is:

```text
16 + length of "Hello"
= 21
```

Capacity and length are different concepts.

```java
sb.length();    // number of characters currently stored
sb.capacity();  // amount of storage currently available
```

Example:

```text
StringBuffer
length   → 5
capacity → 21
```

The capacity can grow automatically when more space is required.

---

# 10. Encapsulation

**Encapsulation** means controlling access to an object's internal state and exposing controlled ways to interact with it.

Instead of allowing outside code to directly modify important data:

```java
class Student {
    private int marks;
}
```

we can provide methods to control access:

```java
class Student {

    private int marks;

    public void setMarks(int marks) {
        this.marks = marks;
    }

    public int getMarks() {
        return marks;
    }
}
```

Usage:

```java
Student student = new Student();

student.setMarks(90);

System.out.println(student.getMarks());
```

The outside code does not directly access:

```java
student.marks
```

because `marks` is private.

---

# 11. `private`

`private` restricts access to the **same class**.

```java
class Student {

    private int marks;

}
```

Code outside `Student` cannot directly access `marks`.

```java
Student s = new Student();

s.marks = 90;   // ❌ not allowed
```

Instead, the class can provide methods that control how the value is accessed or changed.

---

# 12. `this` Keyword

`this` refers to the **current object**.

A common use is when a method parameter and an instance variable have the same name.

```java
class Student {

    private int marks;

    public void setMarks(int marks) {
        this.marks = marks;
    }
}
```

Here there are two different variables named `marks`:

```text
this.marks
    ↓
instance variable

marks
    ↓
method parameter
```

Therefore:

```java
this.marks = marks;
```

means:

```text
current object's marks = parameter marks
```

---

# 13. Why `this` Is Needed

Without `this`:

```java
public void setMarks(int marks) {
    marks = marks;
}
```

both sides refer to the parameter, so the instance variable is not updated.

With `this`:

```java
public void setMarks(int marks) {
    this.marks = marks;
}
```

the distinction is clear.

---

# 14. `this` Depends on the Calling Object

Suppose:

```java
Student s1 = new Student();
Student s2 = new Student();

s1.setMarks(90);
s2.setMarks(80);
```

When:

```java
s1.setMarks(90);
```

executes:

```text
this → s1
```

When:

```java
s2.setMarks(80);
```

executes:

```text
this → s2
```

So `this` always refers to the **current object whose method is being executed**.

---

# Quick Recall

```text
String
→ class
→ reference type
→ immutable

String Pool
→ reuses String literals when possible

StringBuffer
→ mutable
→ synchronized/thread-safe

StringBuilder
→ mutable
→ not synchronized
→ generally preferred when thread safety isn't needed

Encapsulation
→ control access to internal state

private
→ accessible only inside the same class

this
→ reference to the current object

this.marks
→ current object's instance variable

marks
→ method parameter/local variable
```
