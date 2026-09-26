# Java Session 9 — Method Overriding, Packages & Access Modifiers

## 1. Method Overriding

Method overriding occurs when a **subclass provides its own implementation of a method that is already defined in its superclass**.

Example:

```java
class Calculator {

    void add() {
        System.out.println("Calculator add");
    }
}

class AdvancedCalculator extends Calculator {

    @Override
    void add() {
        System.out.println("Advanced Calculator add");
    }
}
```

Now:

```java
AdvancedCalculator calc = new AdvancedCalculator();

calc.add();
```

Output:

```text
Advanced Calculator add
```

The subclass implementation is used instead of the inherited superclass implementation.

---

## 2. Why Method Overriding?

The parent class can provide common/default behavior, while the child class can provide a more specific implementation.

```text
Calculator
    ↓
common add()

AdvancedCalculator
    ↓
specialized add()
```

This allows subclasses to **change or specialize inherited behavior**.

---

## 3. `@Override`

Use the `@Override` annotation when overriding a method:

```java
@Override
void add() {
}
```

It tells the compiler:

> "I intend to override a superclass method."

It also helps catch mistakes such as incorrect method signatures.

---

## 4. Method Overriding vs Method Overloading

These are different concepts.

### Overriding

Same method signature in parent and child:

```java
class A {
    void show() {}
}

class B extends A {
    @Override
    void show() {}
}
```

```text
Parent → show()
Child  → show()
```

### Overloading

Same method name but different parameter lists, usually within the same class:

```java
void add(int a, int b) {}

void add(int a, int b, int c) {}
```

```text
Overriding → inheritance relationship
Overloading → different parameter list
```

---

# 5. Packages

A **package** is a way of grouping related Java classes and interfaces.

Example:

```text
src/
├── com/myapp/
│   ├── Calculator.java
│   └── Student.java
│
└── com/utils/
    ├── MathUtils.java
    └── StringUtils.java
```

A Java file declares its package using:

```java
package com.myapp;
```

Packages help with:

- Organizing code
- Avoiding naming conflicts
- Controlling access between classes

---

# 6. Importing Classes

Suppose:

```java
package com.utils;

public class Calculator {
}
```

A class in another package can import it:

```java
import com.utils.Calculator;
```

Then:

```java
Calculator c = new Calculator();
```

The general syntax is:

```java
import packageName.ClassName;
```

---

## 7. Importing Multiple Classes

You can import all accessible types from a package using `*`:

```java
import com.utils.*;
```

This means:

> Import the accessible types from `com.utils`.

It does **not** recursively import classes from subpackages.

For example:

```text
com.utils
com.utils.math
```

Importing:

```java
import com.utils.*;
```

doesn't automatically import `com.utils.math.*`.

---

# 8. `java.lang` Package

Java automatically makes the `java.lang` package available to every Java source file.

For example:

```java
System.out.println("Hello");
String name = "Sekhar";
Math.max(10, 20);
```

Classes such as:

```text
String
System
Math
Object
```

are in `java.lang`.

Therefore, you don't normally need:

```java
import java.lang.System;
import java.lang.String;
```

---

# 9. `System.out.println()`

`System` is **not a package**.

It is a class:

```text
java.lang
   ↓
System (class)
```

`out` is a static field of `System`, and `println()` is a method of the object referenced by `out`.

So:

```java
System.out.println("Hello");
```

is not:

```text
System → package
```

Instead:

```text
java.lang.System
       ↓
      out
       ↓
   println()
```

---

# 10. Access Modifiers

Java provides access control using access modifiers.

The four main access levels are:

```text
private
default/package-private
protected
public
```

---

# 11. `private`

`private` members can be accessed only from within the same class.

```java
class Student {

    private int marks;

    void display() {
        System.out.println(marks); // ✅
    }
}
```

Another class cannot directly access it:

```java
Student s = new Student();

s.marks = 90; // ❌
```

---

# 12. Default / Package-Private

When you don't specify an access modifier:

```java
class Student {

    int marks;
}
```

`marks` has **default/package-private access**.

It can be accessed by classes in the **same package**.

```text
Same package       → ✅
Different package  → ❌
```

Important:

> Java does not automatically add `public` when you omit the modifier.

No modifier means **package-private**.

---

# 13. `protected`

`protected` is slightly more powerful than default access.

A protected member can be accessed:

1. From classes in the **same package**
2. From subclasses in **different packages**, subject to Java's inheritance/access rules

Example:

```java
class Calculator {

    protected int value;
}
```

A class in the same package can access it:

```java
Calculator c = new Calculator();

System.out.println(c.value); // ✅
```

A subclass in another package can also access the inherited protected member:

```java
class AdvancedCalculator extends Calculator {

    void display() {
        System.out.println(value); // ✅
    }
}
```

### Important cross-package rule

From a different package, `protected` is primarily available through **inheritance**, not as unrestricted access through any `Calculator` object.

So don't simplify it to:

> "Protected means accessible everywhere in subclasses."

Use:

> **Same package OR inherited access from a subclass in another package.**

---

# 14. `public`

`public` provides the broadest access.

```java
public int marks;
```

It can be accessed from classes in other packages, provided the class/member itself is accessible.

---

# 15. Access Modifier Comparison

| Modifier    | Same Class | Same Package | Subclass in Different Package | Other Package |
| ----------- | ---------: | -----------: | ----------------------------: | ------------: |
| `private`   |         ✅ |           ❌ |                            ❌ |            ❌ |
| default     |         ✅ |           ✅ |                            ❌ |            ❌ |
| `protected` |         ✅ |           ✅ |                          ✅\* |            ❌ |
| `public`    |         ✅ |           ✅ |                            ✅ |            ✅ |

`*` Protected access from another package is through inheritance and follows specific Java rules.

---

# Quick Recall

```text
Method overriding
→ Child provides its own implementation
  of an inherited parent method

@Override
→ tells compiler that overriding is intended

Package
→ groups related Java classes/interfaces

import
→ allows use of types from another package

java.lang
→ automatically available

System
→ class, not package

private
→ same class

default
→ same package

protected
→ same package + subclass access across packages

public
→ broadest access
```
