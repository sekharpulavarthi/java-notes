# Java — Interface Types & Lambda Expressions

## 1. Types of Interfaces

### Normal Interface

An interface that can contain **multiple abstract methods**.

```java
interface A {
    void show();
    void display();
}
```

A class implementing it must implement both methods:

```java
class B implements A {

    public void show() {
        System.out.println("show");
    }

    public void display() {
        System.out.println("display");
    }
}
```

> Modern Java interfaces can also contain `default` and `static` methods, so "normal interface = two or more methods" is a simplified learning definition.

---

## 2. Functional Interface

A functional interface has **exactly one abstract method**.

It is also called a:

**SAM — Single Abstract Method interface**

```java
@FunctionalInterface
interface A {
    void show(int i);
}
```

A functional interface can have other `default` or `static` methods; the important rule is that it has **only one abstract method**.

`@FunctionalInterface` tells the compiler to verify this rule.

---

## 3. Marker Interface

A marker interface has **no methods**.

```java
interface Serializable {
}
```

It acts as a signal/metadata to Java or a framework:

> "Objects of this class have this particular capability/permission."

### Example: Serialization

Java provides:

```java
java.io.Serializable
```

If a class implements it:

```java
class Student implements Serializable {
    int rollNo;
    String name;
}
```

it indicates that objects of `Student` are eligible for Java's standard serialization mechanism.

Serialization means converting an object's state into a form that can be stored or transmitted and later reconstructed.

```text
Object
   ↓
Serialization
   ↓
Stored/transmitted data
```

A marker interface itself doesn't perform the serialization; it **marks the class as eligible** for the mechanism.

---

# 4. Lambda Expression

Lambda expressions were introduced in **Java 8**.

Before lambda, an anonymous implementation could look like:

```java
interface A {
    void show(int i);
}
```

```java
A obj = new A() {
    public void show(int i) {
        System.out.println("in show " + i);
    }
};

obj.show(5);
```

With a lambda:

```java
A obj = i -> System.out.println("in show " + i);

obj.show(5);
```

Much less code is required.

---

# 5. Lambda Syntax

General form:

```text
(parameters) -> expression
```

Example:

```java
A obj = (int i) -> System.out.println("in show " + i);
```

The parameter type can usually be inferred:

```java
A obj = i -> System.out.println("in show " + i);
```

Both represent the implementation of:

```java
void show(int i);
```

---

# 6. Lambda with Multiple Parameters

If there are multiple parameters:

```java
interface A {
    void add(int i, int j);
}
```

Lambda:

```java
A obj = (i, j) -> System.out.println(i + j);
```

Multiple parameters require parentheses.

---

# 7. Why Does Lambda Need a Functional Interface?

Consider:

```java
interface A {
    void show(int i);
}
```

and:

```java
A obj = i -> System.out.println(i);
```

Java knows exactly what the lambda represents because `A` has **one abstract method**:

```text
A
 ↓
show(int i)
```

Therefore Java can understand:

```text
i
↓
parameter of show()

System.out.println(i)
↓
implementation of show()
```

If the interface had two abstract methods:

```java
interface A {
    void show(int i);
    void display();
}
```

and we wrote:

```java
A obj = i -> System.out.println(i);
```

Java wouldn't know whether the lambda is supposed to implement `show()` or `display()`.

Therefore:

> **Lambda expressions can be used as implementations of functional interfaces.**

---

# Quick Recall

```text
Normal Interface
→ can have multiple abstract methods

Functional Interface
→ exactly ONE abstract method
→ SAM
→ lambda can be used

Marker Interface
→ no methods
→ acts as a marker/metadata
→ example: Serializable

Lambda
→ introduced in Java 8
→ concise way to provide implementation of a functional interface

Example:

@FunctionalInterface
interface A {
    void show(int i);
}

A obj = i -> System.out.println("in show " + i);

obj.show(5);
```
