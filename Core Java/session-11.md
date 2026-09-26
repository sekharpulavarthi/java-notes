# Java Session 11 — Upcasting, Downcasting, Wrapper Classes & Abstract Classes

## 1. Upcasting

Upcasting means treating a **subclass object as its superclass type**.

```java id="m7y8ab"
class Computer {
    public void display() {
        System.out.println("Computer");
    }
}

class Laptop extends Computer {
    public void display1() {
        System.out.println("Laptop");
    }
}
```

```java id="o0i7kq"
Computer c = new Laptop();
```

Here:

```text id="f3v3cb"
Object created → Laptop
Reference type → Computer
```

A `Laptop` **is a** `Computer`, so this is valid.

### Upcasting is implicit

We don't need to explicitly cast:

```java id="n7xq4v"
Computer c = new Laptop();
```

Java automatically performs the upcasting.

Conceptually:

```text id="0nj2yk"
Laptop object
     ↓
treated as
     ↓
Computer reference
```

---

# 2. What Can We Access After Upcasting?

Suppose:

```java id="xw8q0m"
Computer c = new Laptop();
```

`Laptop` has:

```java id="f4d5g2"
display1()
```

but `Computer` doesn't.

Therefore:

```java id="k7j4pv"
c.display();    // ✅
c.display1();   // ❌
```

Even though the actual object is a `Laptop`, the reference type is `Computer`.

### Important rule

```text id="q6jz9k"
Reference type
→ determines what members you can access at compile time

Actual object type
→ determines which overridden instance method executes at runtime
```

This is the important connection between **upcasting and runtime polymorphism**.

---

# 3. Downcasting

Downcasting means converting a superclass reference back to a subclass reference.

```java id="c8kn4h"
Computer c = new Laptop();

Laptop c1 = (Laptop) c;
```

Here:

```text id="1q5v9w"
Computer reference
       ↓
   downcast
       ↓
Laptop reference
```

Now:

```java id="3m0p4w"
c1.display1();
```

works because `c1` is a `Laptop` reference.

---

# 4. Why Is Explicit Casting Required?

Upcasting is safe because:

```text id="z5qg7k"
Every Laptop is a Computer
```

Therefore:

```java id="74gcq7"
Computer c = new Laptop();
```

is automatically allowed.

But the reverse isn't always safe:

```text id="3sp5t2"
Every Computer is NOT necessarily a Laptop
```

Therefore Java requires explicit casting:

```java id="f9a5dj"
Laptop c1 = (Laptop) c;
```

---

# 5. Downcasting Example

```java id="1n6t7a"
class Computer {

    public void display() {
        System.out.println("This is a Computer");
    }
}

class Laptop extends Computer {

    public void display1() {
        System.out.println("This is a Laptop");
    }
}

public class Demo {

    public static void main(String[] args) {

        Computer c = new Laptop();

        c.display();

        Laptop c1 = (Laptop) c;

        c1.display1();
    }
}
```

The same object is being referred to through two different reference variables:

```text id="h9t3eq"
Stack                         Heap

c  ───────────────────────→ Laptop object
                             ↑
c1 ─────────────────────────┘
```

`c` and `c1` are two references to the **same Laptop object**.

---

# 6. Do We Need a Different Variable for Downcasting?

**No.**

A different variable is not mandatory.

You can write:

```java id="5z8p6k"
Computer c = new Laptop();

c = (Laptop) c;
```

But there is an important issue:

The variable `c` is still declared as type `Computer`.

So even though the object is a `Laptop`, this doesn't change the declared type of `c`:

```java id="2l4q8u"
c.display1(); // ❌
```

The reference variable's declared type remains `Computer`.

Therefore, normally we use:

```java id="5l4f9d"
Laptop c1 = (Laptop) c;
```

when we want to access subclass-specific methods.

---

# 7. Dangerous Downcasting

Downcasting is safe only when the actual object is really an instance of the target subclass.

This is valid:

```java id="c0p0s2"
Computer c = new Laptop();

Laptop l = (Laptop) c; // ✅
```

But:

```java id="xj5n6m"
Computer c = new Computer();

Laptop l = (Laptop) c; // ❌
```

The object is actually a `Computer`, not a `Laptop`.

This results in a `ClassCastException` at runtime.

A common way to check is:

```java id="c7f1b0"
if (c instanceof Laptop) {
    Laptop l = (Laptop) c;
}
```

---

# 8. Wrapper Classes

Java provides a wrapper class for every primitive type.

| Primitive | Wrapper     |
| --------- | ----------- |
| `byte`    | `Byte`      |
| `short`   | `Short`     |
| `int`     | `Integer`   |
| `long`    | `Long`      |
| `float`   | `Float`     |
| `double`  | `Double`    |
| `char`    | `Character` |
| `boolean` | `Boolean`   |

Wrapper classes allow primitive values to be represented as **objects**.

---

# 9. Boxing

Converting a primitive value into its corresponding wrapper object is called **boxing**.

```java id="v5r0kq"
int num = 10;

Integer obj = Integer.valueOf(num);
```

Conceptually:

```text id="4a1d7m"
int
 ↓
Integer object
```

---

# 10. Autoboxing

When Java automatically performs boxing:

```java id="g5n6o8"
int num = 10;

Integer obj = num;
```

Java automatically converts the `int` into an `Integer`.

This is called **autoboxing**.

---

# 11. Unboxing

Converting a wrapper object back to a primitive is called **unboxing**.

```java id="k3r9s2"
Integer obj = 10;

int num = obj.intValue();
```

```text id="0n2xjv"
Integer object
      ↓
     int
```

---

# 12. Auto-unboxing

Java can perform the conversion automatically:

```java id="g6q2e8"
Integer obj = 10;

int num = obj;
```

This is called **auto-unboxing**.

---

# 13. Converting String to `int`

If a number is stored as a `String`:

```java id="1o6p7n"
String value = "100";
```

we can convert it to an `int` using:

```java id="5m8q4r"
int num = Integer.parseInt(value);
```

Now:

```text id="x2l8f4"
"100"  →  100
String    int
```

`parseInt()` is a static method of the `Integer` wrapper class.

---

# 14. Abstract Class

An abstract class is a class that can define a **common structure/contract** for its subclasses.

Example:

```java id="q8v2ka"
abstract class Car {

    abstract void start();
    abstract void stop();
}
```

The methods are declared but don't provide an implementation.

A subclass must provide the implementation:

```java id="p5w6ez"
class Toyota extends Car {

    @Override
    void start() {
        System.out.println("Toyota starts");
    }

    @Override
    void stop() {
        System.out.println("Toyota stops");
    }
}
```

---

# 15. Abstract Method

An abstract method has a declaration but no implementation.

```java id="1x9v3k"
abstract void start();
```

It ends with `;`.

There is no method body.

If a class contains an abstract method, the class **must be abstract**.

```java id="e7s2p1"
abstract class Car {

    abstract void start();
}
```

---

# 16. Abstract Class Does Not Necessarily Need Abstract Methods

An abstract class can contain **zero or more abstract methods**.

For example:

```java id="5j3n7m"
abstract class Car {

    void fuel() {
        System.out.println("Fuel");
    }
}
```

This is valid.

So:

```text id="z5m2w7"
Abstract method exists
        ↓
Class MUST be abstract

Abstract class
        ↓
Does NOT necessarily need an abstract method
```

---

# 17. Abstract Classes Cannot Be Instantiated

You cannot create an object directly from an abstract class:

```java id="r8f0z3"
Car c = new Car(); // ❌
```

Instead, create an object of a concrete subclass:

```java id="g9h4k1"
Car c = new Toyota();
```

This is both:

```text id="2j4p6w"
upcasting
   +
runtime polymorphism
```

The reference is `Car`, but the actual object is `Toyota`.

---

# Quick Recall

```text id="q1w4e7"
Upcasting
→ Child object → Parent reference
→ implicit
→ safe because child IS-A parent

Computer c = new Laptop();

Downcasting
→ Parent reference → Child reference
→ explicit cast required

Laptop l = (Laptop) c;

Downcasting
→ safe only when actual object is the target
  subclass
→ otherwise ClassCastException

Wrapper classes
→ object versions of primitive types

Boxing
→ primitive → wrapper

Autoboxing
→ automatic primitive → wrapper

Unboxing
→ wrapper → primitive

Auto-unboxing
→ automatic wrapper → primitive

Integer.parseInt("100")
→ String → int

Abstract class
→ cannot be instantiated

Abstract method
→ declaration without implementation

Abstract method
→ class must be abstract

Abstract class
→ can contain normal methods
→ does not require an abstract method
```
