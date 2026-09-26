# Java Session 13 — Enums & Annotations

## 1. Enum

`enum` is used when we have a **fixed set of named constants**.

Example:

```java
enum Status {
    Running,
    Failed,
    Pending,
    Success
}
```

Here, `Running`, `Failed`, `Pending`, and `Success` are **enum constants**.

### Important: Are they objects?

**Yes.**

Each enum constant is an **object (instance) of the enum type**.

Conceptually:

```text
Status
 ├── Running  → Status object
 ├── Failed   → Status object
 ├── Pending  → Status object
 └── Success  → Status object
```

Therefore:

```java
Status s = Status.Pending;
```

means:

```text
s
↓
reference to the Pending enum object
```

You don't use `new` to create these objects yourself. Java creates the enum instances automatically.

---

# 2. Why Use Enum?

Suppose an application has a status:

```text
Running
Failed
Pending
Success
```

We could use strings:

```java
String status = "Running";
```

But this allows invalid values:

```java
status = "SomethingRandom";
```

An enum restricts the value to the predefined choices:

```java
Status status = Status.Running;
```

This makes the code safer and clearer.

---

# 3. Enum with `switch`

Enums work very well with `switch`.

```java
enum Status {
    Running,
    Failed,
    Pending,
    Success
}
```

```java
Status s = Status.Pending;

switch (s) {

    case Running:
        System.out.println("All Good");
        break;

    case Failed:
        System.out.println("Try Again");
        break;

    case Pending:
        System.out.println("Please Wait");
        break;

    default:
        System.out.println("Done");
}
```

Inside the switch, we can directly use:

```java
case Running:
```

rather than:

```java
case Status.Running:
```

because the switch expression already tells Java that the type is `Status`.

---

# 4. Enum with `if`

We can also compare enum constants:

```java
if (s == Status.Running) {
    System.out.println("All Good");
}
```

Enum constants can safely be compared using `==`.

---

# 5. `values()`

Every enum provides a `values()` method.

```java
Status[] ss = Status.values();
```

This gives an array containing all enum constants.

Conceptually:

```text
Status.values()

↓
[Running, Failed, Pending, Success]
```

Therefore:

```java
for (Status s : Status.values()) {
    System.out.println(s);
}
```

prints all the constants.

---

# 6. `ordinal()`

Every enum constant has an ordinal position.

```java
Status.Running.ordinal();  // 0
Status.Failed.ordinal();   // 1
Status.Pending.ordinal();  // 2
Status.Success.ordinal();  // 3
```

The numbering starts from `0`.

```text
Running → 0
Failed  → 1
Pending → 2
Success → 3
```

Don't use `ordinal()` as a permanent business ID; it depends on the declaration order.

---

# 7. Enum Is More Powerful Than Simple Constants

An enum can contain:

- variables
- constructors
- methods
- logic

Example:

```java
enum Laptop {

    Macbook(2000),
    XPS(2200),
    Surface,
    ThinkPad(1800);

    private int price;

    private Laptop() {
        price = 500;
    }

    private Laptop(int price) {
        this.price = price;
    }

    public int getPrice() {
        return price;
    }

    public void setPrice(int price) {
        this.price = price;
    }
}
```

This is where enum becomes very interesting.

---

# 8. Are `Macbook`, `XPS`, etc. Objects?

Yes.

These:

```java
Macbook
XPS
Surface
ThinkPad
```

are **four separate instances of `Laptop`**.

Conceptually:

```text
Laptop enum
   │
   ├── Macbook  → Laptop object
   ├── XPS      → Laptop object
   ├── Surface  → Laptop object
   └── ThinkPad → Laptop object
```

So:

```java
Laptop lap = Laptop.Macbook;
```

means `lap` refers to the `Macbook` enum object.

---

# 9. What Happens with `Macbook(2000)`?

Look at:

```java
enum Laptop {

    Macbook(2000),
    XPS(2200),
    Surface,
    ThinkPad(1800);

    ...
}
```

The values in parentheses are **arguments passed to the enum constructor**.

For:

```java
Macbook(2000)
```

Java effectively initializes that enum constant using the constructor:

```java
Laptop(int price)
```

So:

```text
Macbook(2000)
      ↓
Laptop(int price)
      ↓
this.price = 2000
```

Similarly:

```text
XPS(2200)
   ↓
price = 2200

ThinkPad(1800)
   ↓
price = 1800
```

---

# 10. What About `Surface` Without a Price?

We have:

```java
Surface
```

There are no parentheses or arguments.

Therefore Java uses the **no-argument constructor**:

```java
private Laptop() {
    price = 500;
}
```

So:

```text
Macbook(2000) → parameterized constructor → price = 2000
XPS(2200)     → parameterized constructor → price = 2200
Surface       → no-argument constructor   → price = 500
ThinkPad(1800)→ parameterized constructor → price = 1800
```

Therefore:

```java
for (Laptop lap : Laptop.values()) {
    System.out.println(lap + " : " + lap.getPrice());
}
```

produces values equivalent to:

```text
Macbook : 2000
XPS : 2200
Surface : 500
ThinkPad : 1800
```

---

# 11. Are We Creating the Enum Objects Ourselves?

No.

We don't write:

```java
new Laptop(...)
```

for each constant.

The Java runtime creates the enum instances automatically.

We simply declare the constants:

```java
Macbook(2000),
XPS(2200),
Surface,
ThinkPad(1800)
```

Java creates those enum objects.

The constructors are then used to initialize each object.

---

# 12. Why Does `this.price` Work?

Inside:

```java
private Laptop(int price) {
    this.price = price;
}
```

There are two `price`s:

```text
this.price
    ↓
instance variable belonging to the current enum object

price
    ↓
constructor parameter
```

For:

```java
Macbook(2000)
```

the constructor effectively does:

```text
current Macbook object's price = 2000
```

For:

```java
XPS(2200)
```

it does:

```text
current XPS object's price = 2200
```

So each enum object has its own `price`.

---

# 13. `name()`

Every enum constant has a `name()` method.

```java
Laptop.Macbook.name();
```

returns:

```text
"Macbook"
```

Similarly:

```java
Laptop.XPS.name();
```

returns:

```text
"XPS"
```

Inside:

```java
System.out.println("in Laptop " + this.name());
```

`this` refers to the **current enum object**.

So when called by `Macbook`, `this.name()` gives:

```text
Macbook
```

When called by `XPS`, it gives:

```text
XPS
```

---

# 14. Enum and Inheritance

You cannot extend an enum yourself.

```java
class A extends Laptop { } // ❌
```

Enums already have a special inheritance relationship with Java's `Enum` class.

Conceptually:

```text
Object
   ↑
 Enum<Laptop>
   ↑
 Laptop
```

You don't explicitly write:

```java
enum Laptop extends Enum
```

Java handles this automatically.

### Why can't we extend an enum?

Enum types are designed to represent a **fixed set of instances**.

Allowing another class to extend an enum could break that fixed set.

Also, an enum cannot extend another class because Java classes can have only one superclass, and the enum's superclass relationship is already reserved for `Enum`.

---

# 15. Annotations

Annotations provide **metadata/information about code** to the compiler, tools, or runtime.

They start with:

```java
@
```

Examples:

```java
@Override
@Deprecated
```

---

# 16. `@Override`

`@Override` tells the compiler:

> "I intend this method to override a method from the superclass."

Example:

```java
class A {

    public void show() {
        System.out.println("in A show");
    }
}

class B extends A {

    @Override
    public void show() {
        System.out.println("in B show");
    }
}
```

This is useful because the compiler can check whether the method really overrides a superclass method.

---

# 17. Why Is `@Override` Useful?

Suppose we accidentally make a spelling mistake:

```java
class B extends A {

    @Override
    public void showTheDataWhichBelongsToThisClass() {
    }
}
```

If the superclass doesn't contain that exact method signature, the compiler reports an error.

Without `@Override`, Java may treat it as a completely new method.

So:

```text
Without @Override
→ possible accidental new method
→ mistake may be harder to notice

With @Override
→ compiler checks the intention
→ error detected during compilation
```

`@Override` doesn't make the overriding happen. The method would still override if it correctly matches the superclass method.

The annotation simply tells the compiler to **verify your intention**.

---

# 18. `@Deprecated`

`@Deprecated` indicates that a class, method, or other API element **should no longer be used for new code**.

Example:

```java
@Deprecated
class A {
}
```

Or:

```java
class A {

    @Deprecated
    public void oldMethod() {
    }
}
```

When code uses a deprecated API, the compiler/IDE can warn you.

It generally means:

```text
"This still exists, but you should use something else instead."
```

---

# Quick Recall

```text
enum
→ fixed set of named constants

enum constants
→ actual objects/instances of the enum type

Status.Pending
→ reference to the Pending enum object

values()
→ returns all enum constants

ordinal()
→ declaration position starting from 0

Enum can contain
→ variables
→ constructors
→ methods

Macbook(2000)
→ enum constant
→ object created automatically
→ 2000 passed to constructor

Surface
→ no argument
→ no-argument constructor
→ price = 500

this
→ current enum object

this.name()
→ name of current enum constant

enum inheritance
→ enum implicitly extends java.lang.Enum
→ cannot extend another class
→ cannot be extended by your class

Annotation
→ metadata/information about code

@Override
→ compiler verifies that method actually overrides

@Deprecated
→ tells developers that an API should generally no longer be used
```
