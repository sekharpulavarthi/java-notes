# Java Session 12 — Inner Classes, Anonymous Classes & Interfaces

## 1. Inner Class

An **inner class** is a class defined inside another class.

```java
class A {

    class B {
        void config() {
            System.out.println("in config");
        }
    }
}
```

Here:

```text
A → outer class
B → inner class
```

### Why use an inner class?

Use an inner class when a class is **closely related to, and mainly used by, its outer class**.

It helps keep related implementation details together instead of exposing a separate top-level class.

---

## 2. Non-static Inner Class

A normal inner class belongs to an **instance of the outer class**.

```java
A obj = new A();

A.B obj1 = obj.new B();

obj1.config();
```

The important part is:

```java
obj.new B()
```

Conceptually:

```text
A object
   ↓
contains/accesses
   ↓
B inner-class object
```

You need an outer-class object to create a non-static inner-class object.

---

## 3. Static Nested Class

If the inner class is declared `static`:

```java
class A {

    static class B {

        void config() {
            System.out.println("in config");
        }
    }
}
```

You don't need an object of `A` to create `B`.

```java
A.B obj1 = new A.B();

obj1.config();
```

Conceptually:

```text
A.B
 ↑
B belongs to the class A,
not to a particular A object.
```

### Remember

```text
Non-static inner class
→ needs outer-class object

Static nested class
→ does NOT need outer-class object
→ accessed using OuterClass.InnerClass
```

> Strictly speaking, Java commonly calls the `static` version a **static nested class**, rather than an "inner class."

---

# 4. Anonymous Inner Class

An **anonymous class** is a class without a name, created for a specific one-time use.

Suppose:

```java
class A {

    public void show() {
        System.out.println("in A show");
    }
}
```

Normally, if we want a subclass that only changes `show()`:

```java
class B extends A {

    @Override
    public void show() {
        System.out.println("in B show");
    }
}
```

Then:

```java
A obj = new B();
```

But if `B` is needed **only once**, creating a separate named class can be unnecessary.

We can use an anonymous class:

```java
A obj = new A() {

    @Override
    public void show() {
        System.out.println("in new show");
    }
};

obj.show();
```

Here:

```text
new A()
   ↓
create A subclass anonymously
   ↓
override show()
   ↓
create its object
```

There is no class name such as `B`.

---

# 5. Anonymous Class with Abstract Class

An abstract class cannot be directly instantiated:

```java
abstract class A {

    abstract void show();
}
```

This is invalid:

```java
A obj = new A(); // ❌
```

But we can create an anonymous subclass:

```java
A obj = new A() {

    @Override
    public void show() {
        System.out.println("in new show");
    }
};
```

This is valid.

We are **not creating an object of the abstract class directly**.

We are creating an anonymous concrete subclass of `A` and an object of that subclass.

Conceptually:

```text
abstract A
   ↑
   │ extends
anonymous subclass
   ↓
object created
```

Useful when the abstract class needs to be implemented **only once**.

---

# 6. Interface

An interface defines a **contract** — it describes what a class must provide without requiring that class to inherit implementation from the interface.

Example:

```java
interface Computer {

    void code();
}
```

A class implements it:

```java
class Laptop implements Computer {

    public void code() {
        System.out.println("Laptop coding");
    }
}
```

Another class can also implement it:

```java
class Desktop implements Computer {

    public void code() {
        System.out.println("Desktop coding faster");
    }
}
```

---

# 7. Interface Methods

In a traditional/basic interface example:

```java
interface Computer {

    void code();
}
```

The method is implicitly:

```java
public abstract void code();
```

So:

```java
void code();
```

means:

```text
public + abstract
```

Modern Java interfaces can also contain `default`, `static`, and `private` methods with implementations, so "all interface methods are abstract" is a simplification.

---

# 8. Interface Variables

Variables declared in an interface are implicitly:

```text
public static final
```

Example:

```java
interface Computer {

    int MAX = 100;
}
```

is equivalent to:

```java
public static final int MAX = 100;
```

Therefore:

```java
System.out.println(Computer.MAX);
```

works.

But:

```java
Computer.MAX = 200; // ❌
```

doesn't work because the variable is `final`.

### Why `static final`?

The value belongs to the **interface itself**, not to individual objects, and it cannot be changed.

```text
Interface
   ↓
shared constant
   ↓
cannot be reassigned
```

---

# 9. Does an Interface Have Objects in the Heap?

An interface itself is **not instantiated**:

```java
Computer c = new Computer(); // ❌
```

The interface defines a type/contract.

The actual object is created from a concrete implementing class:

```java
Computer c = new Laptop();
```

Memory conceptually:

```text
Stack                    Heap

c ───────────────────→ Laptop object
                         ↑
                    implements
                    Computer
```

The interface reference doesn't create a separate "Computer object" in the heap.

The `Laptop` object is the actual object.

---

# 10. Multiple Interfaces

A class can implement multiple interfaces.

```java
interface A {
    void methodA();
}

interface B {
    void methodB();
}

class C implements A, B {

    public void methodA() {}
    public void methodB() {}
}
```

This is one of the important differences from class inheritance:

```text
class → extends → one superclass

class → implements → multiple interfaces
```

---

# 11. Interface Can Extend Another Interface

An interface can extend another interface.

```java
interface A {
    void methodA();
}

interface B extends A {
    void methodB();
}
```

A class implementing `B` must satisfy the inherited contract from `A` as well.

---

# 12. Why Do We Need Interfaces?

Consider:

```text
Computer
   ↑
   │
 ┌───────┐
 │       │
Laptop  Desktop
```

Both Laptop and Desktop are computers, but their implementations can be different.

```java
interface Computer {

    void code();
}
```

```java
class Laptop implements Computer {

    public void code() {
        System.out.println("Code, compile, run");
    }
}
```

```java
class Desktop implements Computer {

    public void code() {
        System.out.println("Code, compile faster");
    }
}
```

Now the developer doesn't need to care about the exact implementation.

```java
class Developer {

    public void devApp(Computer computer) {
        computer.code();
    }
}
```

We can pass either:

```java
Computer computer = new Laptop();

Developer developer = new Developer();
developer.devApp(computer);
```

or:

```java
Computer computer = new Desktop();

developer.devApp(computer);
```

The `Developer` works with the **Computer contract**, not with a specific Laptop or Desktop implementation.

---

# 13. Interface + Runtime Polymorphism

This is the important connection:

```java
Computer c = new Laptop();
```

or:

```java
Computer c = new Desktop();
```

The reference type is:

```text
Computer
```

but the actual object can be:

```text
Laptop
```

or:

```text
Desktop
```

Therefore:

```java
c.code();
```

executes the implementation belonging to the actual object.

```text
Computer reference
        ↓
 ┌──────┴──────┐
 ↓             ↓
Laptop       Desktop
object       object
 ↓             ↓
Laptop.code  Desktop.code
```

This gives us **runtime polymorphism**.

---

# 14. Abstract Class vs Interface — Basic Mental Model

### Abstract class

Use when you want to define a **common base class** and potentially share state/implementation.

```java
abstract class Computer {

    abstract void code();

    void start() {
        System.out.println("Starting");
    }
}
```

### Interface

Use when you primarily want to define a **contract/capability** that different, potentially unrelated classes can implement.

```java
interface Computer {

    void code();
}
```

A class can implement multiple interfaces:

```java
class Laptop implements Computer, Portable {
}
```

---

# Quick Recall

```text
Inner class
→ class inside another class
→ useful when tightly related to outer class

Non-static inner class
→ needs outer-class object

Static nested class
→ doesn't need outer-class object

Anonymous class
→ class without a name
→ useful for one-time implementation

Abstract class + anonymous class
→ creates an anonymous concrete subclass
→ not a direct abstract-class object

Interface
→ defines a contract/type

implements
→ class → interface

extends
→ class → class
→ interface → interface

Interface variable
→ public static final
→ constant
→ cannot be reassigned

Interface itself
→ cannot be instantiated

Computer c = new Laptop();
→ interface reference
→ Laptop object

Computer c = new Desktop();
→ interface reference
→ Desktop object

One class
→ can implement multiple interfaces

Interface
→ can extend another interface
```
