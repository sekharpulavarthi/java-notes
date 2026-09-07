# Java Session 10 — Polymorphism, Dynamic Method Dispatch & `final`

## 1. Polymorphism

**Poly** → many
**Morphism** → forms/behavior

Polymorphism means **one interface/reference can represent different forms of behavior**.

Two common types:

```text
Compile-time polymorphism → Method Overloading
Runtime polymorphism      → Method Overriding
```

---

# 2. Compile-Time Polymorphism — Method Overloading

When multiple methods have the **same name but different parameter lists**, the compiler determines which method should be called based on the arguments.

```java
class Calculator {

    int add(int a, int b) {
        return a + b;
    }

    int add(int a, int b, int c) {
        return a + b + c;
    }
}
```

```java
Calculator c = new Calculator();

c.add(10, 20);       // add(int, int)
c.add(10, 20, 30);   // add(int, int, int)
```

The method to invoke is determined at **compile time**.

```text
Same method name
       ↓
Different parameters
       ↓
Compiler identifies the matching method
```

---

# 3. Runtime Polymorphism — Method Overriding

Runtime polymorphism occurs when a subclass overrides a method of its superclass.

```java
class Computer {

    public void display() {
        System.out.println("This is a Computer");
    }
}

class Laptop extends Computer {

    @Override
    public void display() {
        System.out.println("This is a Laptop");
    }
}
```

The subclass provides a different implementation of the inherited method.

---

# 4. Dynamic Method Dispatch

Dynamic method dispatch is the mechanism through which Java determines **at runtime which overridden method should execute**.

Example:

```java
Computer c = new Laptop();

c.display();
```

There are two different types involved:

```text
Reference type → Computer
Object type    → Laptop
```

```text
Computer c
    ↓
    reference
    ↓
Laptop object
```

Even though the reference is of type `Computer`, the actual object is a `Laptop`.

Therefore:

```java
c.display();
```

calls:

```text
Laptop.display()
```

not:

```text
Computer.display()
```

---

# 5. Why Is This Possible?

Because:

> **A Laptop is a Computer.**

Therefore, a parent-class reference can refer to a child-class object.

```java
Computer c = new Laptop();
```

This is called **upcasting**.

```text
Computer
    ↑
    │
Laptop
```

A `Laptop` object can be treated as a `Computer`.

---

# 6. Important Rule

For overridden instance methods:

> The method that executes is determined by the **actual object**, not simply by the reference type.

Example:

```java
Computer c = new Laptop();

c.display();
```

```text
Reference type → Computer
Object type    → Laptop

Result → Laptop.display()
```

---

# 7. Changing the Object Behind a Reference

Consider:

```java
public class Demo {

    public static void main(String[] args) {

        Computer c = new Laptop();

        c.display();

        c = new Computer();

        c.display();
    }
}
```

Output:

```text
This is a Laptop
This is a Computer
```

### First statement

```java
Computer c = new Laptop();
```

A `Laptop` object is created in the **heap**.

The reference variable `c` exists in the relevant stack frame and stores a reference to that object.

```text
Stack                    Heap

c ───────────────────→ Laptop object
                         display()
```

Calling:

```java
c.display();
```

uses the object currently referenced by `c`.

Therefore:

```text
Laptop.display()
```

executes.

---

### Then:

```java
c = new Computer();
```

A **new Computer object** is created in the heap.

The reference stored in `c` is updated to refer to the new object.

Conceptually:

```text
Before:

Stack                    Heap

c ───────────────────→ Laptop object


After:

Stack                    Heap

c ───────────────────→ Computer object

                         Laptop object
                         ↑
                         no longer referenced
```

The old `Laptop` object doesn't disappear immediately. If it becomes unreachable and is no longer needed, it can eventually be **garbage collected**.

Then:

```java
c.display();
```

uses the new `Computer` object:

```text
Computer.display()
```

---

# 8. Stack vs Heap in This Example

Keep the simplified mental model:

### Stack

Contains method execution frames and local variables/reference variables.

```text
main()
 └── c → reference to object
```

### Heap

Contains objects created using `new`.

```text
new Laptop()
     ↓
Laptop object → Heap

new Computer()
     ↓
Computer object → Heap
```

When:

```java
c = new Computer();
```

the variable `c` doesn't move to another place.

Instead, **the reference stored in `c` changes to point to the new object**.

---

# 9. Runtime Polymorphism Requires Inheritance

Dynamic method dispatch works when there is a parent-child relationship.

```java
Computer c = new Laptop();
```

because:

```text
Laptop extends Computer
```

Without an inheritance relationship, this assignment isn't possible:

```java
Computer c = new Car(); // ❌
```

---

# 10. `final` Keyword

`final` can be applied to:

```text
Variable
Method
Class
```

Each has a different purpose.

---

## 10.1 Final Variable

A `final` variable cannot be reassigned after it has been initialized.

```java
final int x = 10;

x = 20; // ❌
```

Think:

```text
final variable
     ↓
cannot be reassigned
```

---

## 10.2 Final Method

A `final` method cannot be overridden by a subclass.

```java
class Calculator {

    final void display() {
        System.out.println("Calculator");
    }
}
```

This is not allowed:

```java
class AdvancedCalculator extends Calculator {

    @Override
    void display() {   // ❌
    }
}
```

Use `final` when the superclass wants to prevent subclasses from changing a method's implementation.

---

## 10.3 Final Class

A `final` class cannot be extended.

```java
final class Calculator {
}
```

This is not allowed:

```java
class AdvancedCalculator extends Calculator { // ❌
}
```

```text
final class
     ↓
cannot be inherited
```

---

# 11. `Object` Class and `toString()`

Every Java class ultimately inherits from the `Object` class.

For example:

```java
class Student {
}
```

Conceptually:

```text
Object
   ↑
Student
```

`Object` provides commonly used methods such as:

```text
toString()
equals()
hashCode()
```

among others.

---

# 12. What Happens When We Print an Object?

Suppose:

```java
class Student {

    int rollno;
    String name;
}
```

And:

```java
Student s = new Student();

System.out.println(s);
```

Java converts the object to a string representation by invoking its `toString()` method.

Since `Student` hasn't overridden `toString()`, the inherited implementation from `Object` is used.

You may see something similar to:

```text
Student@5e2de80c
```

The exact value isn't important.

It is a default object representation involving the class name and hash-code-related information.

---

# 13. Overriding `toString()`

We can provide our own implementation:

```java
class Student {

    int rollno;
    String name;

    @Override
    public String toString() {
        return rollno + " : " + name;
    }
}
```

Now:

```java
Student s = new Student();

s.rollno = 1;
s.name = "Sekhar";

System.out.println(s);
```

can produce:

```text
1 : Sekhar
```

Instead of the default representation.

---

# Quick Recall

```text
Polymorphism
→ one thing, many forms/behaviors

Compile-time polymorphism
→ method overloading
→ compiler chooses method

Runtime polymorphism
→ method overriding
→ actual object determines overridden method

Dynamic method dispatch
→ runtime selection of overridden method

Computer c = new Laptop();

Reference type → Computer
Object type    → Laptop

c.display()
→ Laptop.display()

c = new Computer()
→ new Computer object created in heap
→ c now refers to new object
→ old object may become eligible for GC

final variable
→ cannot be reassigned

final method
→ cannot be overridden

final class
→ cannot be extended

Object
→ ultimate superclass of Java classes

toString()
→ inherited from Object
→ can be overridden to provide a meaningful object representation
```
