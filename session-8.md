# Java Day 8 — Anonymous Objects & Inheritance

## 1. Anonymous Object

An object can be created without storing its reference in a variable.

```java
new Calculator().add(10, 20);
```

Here, the `Calculator` object is created, but there is no reference variable pointing to it.

This is called an **anonymous object**.

```text
Normal object:

Calculator c = new Calculator();
        ↓
    reference
        ↓
      object

Anonymous object:

new Calculator()
       ↓
     object
```

### Key point

An anonymous object is useful when the object is needed only once and does not need to be reused.

```java
new Calculator().add(10, 20);
```

After the expression is completed, there is no reference variable through which you can directly access that particular object again.

---

# 2. Inheritance

**Inheritance** allows one class to reuse the properties and behavior of another class.

Syntax:

```java
class AdvancedCalculator extends Calculator {
}
```

Here:

```text
Calculator
    ↑
    │ extends
    │
AdvancedCalculator
```

- `Calculator` → **superclass / parent class**
- `AdvancedCalculator` → **subclass / child class**

The subclass can use accessible members of the superclass.

---

# 3. Why Do We Need Inheritance?

Suppose:

```java
class Calculator {

    int add(int a, int b) {
        return a + b;
    }

    int subtract(int a, int b) {
        return a - b;
    }
}
```

Now we want an advanced calculator:

```java
class AdvancedCalculator extends Calculator {

    int multiply(int a, int b) {
        return a * b;
    }
}
```

We don't need to rewrite `add()` and `subtract()`.

```java
AdvancedCalculator calc = new AdvancedCalculator();

calc.add(10, 20);
calc.subtract(20, 10);
calc.multiply(10, 20);
```

The subclass gets the accessible functionality of the superclass.

### Main benefit

```text
Inheritance
→ code reuse
→ reduces duplication
→ represents an "is-a" relationship
```

Example:

```text
Porsche is a Car
AdvancedCalculator is a Calculator
Dog is an Animal
```

---

# 4. `extends`

The `extends` keyword establishes inheritance between classes.

```java
class AdvancedCalculator extends Calculator {
}
```

Meaning:

> `AdvancedCalculator` inherits from `Calculator`.

Java allows a class to directly extend **one class**.

---

# 5. Multilevel Inheritance

A class can extend another subclass.

```java
class Calculator {
}

class AdvancedCalculator extends Calculator {
}

class VeryAdvancedCalculator extends AdvancedCalculator {
}
```

The inheritance chain becomes:

```text
Calculator
     ↑
AdvancedCalculator
     ↑
VeryAdvancedCalculator
```

This is called **multilevel inheritance**.

---

# 6. Single-Level Inheritance

When one class directly extends another class:

```java
class Calculator {
}

class AdvancedCalculator extends Calculator {
}
```

This is **single-level inheritance**.

---

# 7. Multiple Inheritance

Java does **not** allow a class to extend multiple classes.

This is not allowed:

```java
class C extends A, B {   // ❌
}
```

### Why?

Suppose both parents have the same method:

```java
class A {
    void show() {
        System.out.println("A");
    }
}

class B {
    void show() {
        System.out.println("B");
    }
}
```

If:

```text
C extends A and B
```

and we call:

```java
c.show();
```

there is ambiguity:

```text
Should show() come from A?
or
Should show() come from B?
```

Java avoids this ambiguity by not allowing multiple class inheritance.

---

# 8. `super()`

When a subclass object is created, the superclass constructor is also involved.

Example:

```java
class Calculator {

    Calculator() {
        System.out.println("Calculator constructor");
    }
}

class AdvancedCalculator extends Calculator {

    AdvancedCalculator() {
        System.out.println("Advanced Calculator constructor");
    }
}
```

When:

```java
AdvancedCalculator ac = new AdvancedCalculator();
```

the superclass constructor executes before the subclass constructor.

Conceptually:

```text
new AdvancedCalculator()
        ↓
superclass constructor
        ↓
subclass constructor
```

---

# 9. Implicit `super()`

If you don't explicitly write a `super(...)` call in a constructor, Java implicitly inserts:

```java
super();
```

Example:

```java
class AdvancedCalculator extends Calculator {

    AdvancedCalculator() {
        // compiler implicitly calls super();
        System.out.println("Advanced");
    }
}
```

Conceptually:

```java
AdvancedCalculator() {
    super();
    System.out.println("Advanced");
}
```

`super()` calls the **no-argument constructor of the superclass**.

---

# 10. Calling a Parameterized Superclass Constructor

Suppose:

```java
class Calculator {

    Calculator(int value) {
        System.out.println(value);
    }
}
```

Then:

```java
class AdvancedCalculator extends Calculator {

    AdvancedCalculator(int value) {
        super(value);
    }
}
```

Here:

```java
super(value);
```

calls the parameterized constructor of the superclass.

### Important

If the superclass has only:

```java
Calculator(int value)
```

then:

```java
super();
```

will not work because there is no no-argument superclass constructor.

---

# 11. Constructor Execution Order

Example:

```java
class A {

    A() {
        System.out.println("A");
    }
}

class B extends A {

    B() {
        System.out.println("B");
    }
}

class C extends B {

    C() {
        System.out.println("C");
    }
}
```

Creating:

```java
C obj = new C();
```

results in:

```text
A
B
C
```

The superclass constructor executes before the subclass constructor.

---

# 12. `this()` vs `super()`

These are easy to confuse.

### `super()`

Calls a constructor of the **parent class**.

```java
super();
```

```text
Current class
     ↓
  super()
     ↓
Parent class constructor
```

### `this()`

Calls another constructor of the **same class**.

```java
this();
```

```text
Current constructor
       ↓
     this()
       ↓
Another constructor
of the same class
```

Example:

```java
class Student {

    Student() {
        System.out.println("Default");
    }

    Student(int marks) {
        this();
        System.out.println("Parameterized");
    }
}
```

When:

```java
Student s = new Student(90);
```

execution is:

```text
Student(int)
    ↓
this()
    ↓
Student()
```

---

# 13. `Object` Class

Java has a top-level class called:

```java
Object
```

Every class in Java ultimately inherits from `Object`, either directly or through another superclass.

Example:

```java
class Calculator {
}
```

Conceptually:

```text
Object
   ↑
Calculator
```

If:

```java
class AdvancedCalculator extends Calculator {
}
```

then:

```text
Object
   ↑
Calculator
   ↑
AdvancedCalculator
```

Therefore, methods defined in `Object` are ultimately available to Java objects, subject to their access and overriding rules.

---

# Quick Recall

```text
Anonymous object
→ object created without storing its reference

extends
→ establishes inheritance

Superclass
→ parent class

Subclass
→ child class

Inheritance
→ reuse functionality from another class
→ "is-a" relationship

Single-level
→ A → B

Multilevel
→ A → B → C

Multiple class inheritance
→ not supported in Java

super()
→ calls superclass constructor

super(value)
→ calls parameterized superclass constructor

this()
→ calls another constructor in the same class

Object
→ root superclass of Java's class hierarchy
```
