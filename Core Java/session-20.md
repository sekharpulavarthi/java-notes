### Session: Local Variable Type Inference, Sealed Classes & Records

````markdown
# Java — Local Variable Type Inference, Sealed Classes & Records

## 1. Local Variable Type Inference (`var`)

Java 10 introduced **local variable type inference** using the `var` keyword.

Before Java 10:

```java
int x = 10;
double price = 99.99;
String name = "Sekhar";
```
````

With `var`:

```java
var x = 10;
var price = 99.99;
var name = "Sekhar";
```

The compiler determines the type of the variable from the initializer.

```java
var x = 10;          // int
var price = 10.5;    // double
var name = "Java";   // String
```

### Important: `var` does NOT mean dynamically typed

Java is still a **statically typed language**.

The compiler determines the type at compile time.

```java
var x = 10;

x = 20;       // valid
x = "Hello";  // compilation error
```

The compiler effectively knows that `x` is an `int`.

`var` is only syntactic convenience; it does not change Java's type system.

---

## 2. `var` With Objects

`var` can also be used when creating objects.

```java
var person = new Person();
```

The compiler infers:

```java
Person person = new Person();
```

It can also be used with collections and other reference types:

```java
var list = new ArrayList<String>();
var map = new HashMap<Integer, String>();
```

The inferred type is still the actual declared compile-time type.

---

## 3. `var` Must Have an Initializer

The compiler needs an initializer to determine the type.

Valid:

```java
var x = 10;
var name = "Java";
var person = new Person();
```

Invalid:

```java
var x;
```

This causes a compilation error because the compiler cannot determine the type of `x`.

A local variable declared with `var` must have an initializer.

---

## 4. Where Can `var` Be Used?

`var` can be used for **local variables**, including:

- Local variables inside methods
- Variables declared inside blocks
- Variables declared in loops

Example:

```java
public void test() {

    var x = 10;

    if (x > 5) {
        var message = "Greater than 5";
    }
}
```

`var` cannot be used for:

- Instance variables
- Static/class variables
- Method parameters
- Method return types
- Fields

Invalid:

```java
class Example {

    var value = 10;       // Compilation error

    static var count = 5; // Compilation error
}
```

The reason is that `var` represents **local variable type inference**, not a general-purpose type declaration.

---

## 5. `var` and Memory

Consider:

```java
public void test() {
    var x = 10;
}
```

`x` is a **local variable** because it is declared inside a method.

The important distinction is:

- `x` has local-variable scope.
- It is not an instance variable.
- It is not stored as part of the object's instance state.

Conceptually, local variables and their references are associated with the current method's execution/frame, while objects created with `new` are allocated on the heap.

For example:

```java
public void test() {
    var person = new Person();
}
```

Here:

- `person` is a local reference variable.
- The `Person` object created using `new` is a heap object.
- `person` refers to that object.

A simplified conceptual view:

```text
Stack / current method frame
----------------------------
person  -----------+
                   |
                   v
Heap
----------------------------
Person object
```

However, JVM implementations are more sophisticated than this simplified stack/heap model. The exact memory behavior is determined by the JVM implementation and optimizations such as escape analysis.

Also, Java does not use a "queue" for storing these variables in the sense being discussed here.

---

## 6. `var` With Arrays

`var` can be used with arrays.

```java
int[] arr = new int[10];
```

can be written as:

```java
var arr = new int[10];
```

The compiler infers:

```java
int[]
```

So:

```java
var arr = new int[10];
arr[0] = 100;
```

is valid.

### Array declaration syntax

Do not write:

```java
var[] arr = new int[10]; // Invalid
```

`var` itself is not used with `[]` to indicate the inferred array type.

Instead, the array type is inferred from the initializer:

```java
var arr = new int[10];
```

---

## 7. `var` With Array Initializers

This is valid:

```java
var arr = new int[] {10, 20, 30};
```

The compiler infers:

```java
int[]
```

But this is invalid:

```java
var arr = {10, 20, 30};
```

The array initializer by itself does not provide enough type information for `var`.

---

## 8. `var` Does Not Work With `null`

This is invalid:

```java
var value = null;
```

The compiler cannot infer a specific type from `null`.

---

# 9. Sealed Classes

Java 17 introduced **sealed classes** as a permanent feature.

A sealed class allows us to explicitly control which classes are allowed to directly extend it.

Syntax:

```java
public sealed class Vehicle
        permits Car, Bike {
}
```

Only the classes listed in `permits` can directly extend `Vehicle`.

```java
public final class Car extends Vehicle {
}

public final class Bike extends Vehicle {
}
```

Trying to extend `Vehicle` with an unpermitted class results in a compilation error:

```java
public class Truck extends Vehicle {
}
```

---

## 10. Why Use Sealed Classes?

Suppose we have:

```java
abstract class Payment {
}
```

Any class can potentially extend it.

With a sealed class:

```java
sealed class Payment
        permits CreditCardPayment, UPIPayment {
}
```

we explicitly define the permitted direct subclasses.

This is useful when the hierarchy should be controlled and known.

---

# 11. A Permitted Subclass Must Be `final`, `sealed`, or `non-sealed`

If a class directly extends a sealed class, it must declare one of:

- `final`
- `sealed`
- `non-sealed`

Example:

```java
public sealed class Vehicle
        permits Car, Bike, Truck {
}
```

### Option 1: `final`

```java
public final class Car extends Vehicle {
}
```

No class can extend `Car`.

```text
Vehicle
   |
  Car (final)
```

---

### Option 2: `sealed`

```java
public sealed class Bike extends Vehicle
        permits SportsBike {
}
```

Now only the permitted class can directly extend `Bike`.

```java
public final class SportsBike extends Bike {
}
```

Hierarchy:

```text
Vehicle
   |
  Bike (sealed)
   |
SportsBike (final)
```

---

### Option 3: `non-sealed`

```java
public non-sealed class Truck extends Vehicle {
}
```

Once a subclass is declared `non-sealed`, the sealing restriction is removed from that branch.

Therefore:

```java
public class HeavyTruck extends Truck {
}
```

is valid.

Hierarchy:

```text
Vehicle (sealed)
   |
 Truck (non-sealed)
   |
HeavyTruck
```

---

# 12. Sealed Classes With `extends` and `implements`

A sealed class can extend another class and implement interfaces.

Example:

```java
public sealed class Car
        extends Vehicle
        implements Drivable
        permits SportsCar, ElectricCar {
}
```

The class declaration follows the normal Java order:

```text
class
    ↓
extends
    ↓
implements
    ↓
permits
```

So `permits` comes after `extends` and `implements`.

---

# 13. Sealed Interfaces

Interfaces can also be sealed.

```java
public sealed interface Payment
        permits CreditCardPayment, UPIPayment {
}
```

A permitted interface implementation must also be declared as:

- `final`
- `sealed`
- `non-sealed`

Example:

```java
public final class CreditCardPayment
        implements Payment {
}
```

---

## 14. Sealed Interface Extending Another Sealed Interface

Interfaces cannot be declared `final`.

For example:

```java
public sealed interface X
        permits Y {
}
```

If `Y` directly extends `X`, then `Y` must be either:

- `sealed`, or
- `non-sealed`

Example using `sealed`:

```java
public sealed interface X
        permits Y {
}

public sealed interface Y
        extends X
        permits Z {
}

public final class Z implements Y {
}
```

Example using `non-sealed`:

```java
public sealed interface X
        permits Y {
}

public non-sealed interface Y
        extends X {
}
```

Now other interfaces can extend `Y`:

```java
public interface Z extends Y {
}
```

### Key point

A permitted direct subtype of a sealed type must be:

```text
final
sealed
non-sealed
```

For classes, all three are possible.

For interfaces, `final` is not possible, so a permitted interface subtype must be:

```text
sealed
or
non-sealed
```

---

# 15. Records

Java 16 introduced **records** as a permanent feature.

A record is a concise way to create a class whose primary purpose is to model and carry data.

Without a record, a simple data class can require:

- private fields
- constructor
- getters
- `equals()`
- `hashCode()`
- `toString()`

Example:

```java
public class Person {

    private final String name;
    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String name() {
        return name;
    }

    public int age() {
        return age;
    }

    // equals()
    // hashCode()
    // toString()
}
```

With a record:

```java
public record Person(String name, int age) {
}
```

This single declaration provides the standard data-oriented behavior.

---

# 16. Record Components

In:

```java
public record Person(String name, int age) {
}
```

`name` and `age` are called **record components**.

The record automatically provides accessor methods:

```java
Person person = new Person("Sekhar", 27);

System.out.println(person.name());
System.out.println(person.age());
```

Notice that record accessors are:

```java
person.name()
person.age()
```

not:

```java
person.getName()
person.getAge()
```

---

# 17. Records Are Immutable Data Carriers

Record components are private and final in the generated representation.

Therefore, this is not allowed:

```java
Person person = new Person("Sekhar", 27);

person.age = 28; // Invalid
```

There is no setter generated for a record component.

A record is designed to model data whose component references cannot be reassigned after construction.

For example:

```java
public record Person(String name, int age) {
}
```

The reference stored for `name` and the value stored for `age` cannot be reassigned after construction.

### Important

"Record is immutable" does not automatically mean that every object reachable through a record component is deeply immutable.

For example:

```java
public record Team(List<String> members) {
}
```

The record's `members` reference cannot be reassigned, but the underlying `List` could still be mutable.

---

# 18. `toString()` in Records

Normal classes inherit `toString()` from `Object` unless they override it.

The default `Object.toString()` produces a string containing the class name and a hash-code-based representation, which is often perceived as a memory address, but it should not be described as an actual memory address.

Records automatically provide a useful `toString()` implementation.

Example:

```java
public record Person(String name, int age) {
}
```

```java
Person p = new Person("Sekhar", 27);

System.out.println(p);
```

Output will be similar to:

```text
Person[name=Sekhar, age=27]
```

So we do not need to manually override `toString()` for the normal record representation.

---

# 19. `equals()` and `hashCode()` in Records

Normal classes inherit `equals()` from `Object` unless they override it.

`Object.equals()` compares object identity by default.

Example:

```java
Person p1 = new Person("Sekhar", 27);
Person p2 = new Person("Sekhar", 27);
```

For a normal class that does not override `equals()`:

```java
System.out.println(p1.equals(p2));
```

would normally be:

```text
false
```

because they are two different objects.

Records automatically provide value-based implementations of:

```java
equals()
hashCode()
```

Therefore:

```java
public record Person(String name, int age) {
}
```

```java
Person p1 = new Person("Sekhar", 27);
Person p2 = new Person("Sekhar", 27);

System.out.println(p1.equals(p2));
```

prints:

```text
true
```

because their record components have equal values.

---

# 20. Record Constructor

A record automatically gets a **canonical constructor** corresponding to all record components.

Example:

```java
public record Person(String name, int age) {
}
```

Conceptually, it has a constructor equivalent to:

```java
public Person(String name, int age) {
    this.name = name;
    this.age = age;
}
```

We normally don't need to write this constructor ourselves.

---

# 21. Explicit Canonical Constructor

We can define the canonical constructor ourselves when validation or additional logic is required.

Example:

```java
public record Person(String name, int age) {

    public Person(String name, int age) {
        if (age < 0) {
            throw new IllegalArgumentException("Age cannot be negative");
        }

        this.name = name;
        this.age = age;
    }
}
```

The parameter list must correspond to all record components.

---

# 22. Compact Canonical Constructor

A compact canonical constructor is a shorter way to write the canonical constructor.

Example:

```java
public record Person(String name, int age) {

    public Person {
        if (age < 0) {
            throw new IllegalArgumentException("Age cannot be negative");
        }
    }
}
```

We don't explicitly write:

```java
this.name = name;
this.age = age;
```

The compiler handles the assignment of the constructor parameters to the record components.

This is especially useful for validation.

Example:

```java
public record Employee(String name, int age) {

    public Employee {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name cannot be empty");
        }

        if (age < 18) {
            throw new IllegalArgumentException("Employee must be at least 18");
        }
    }
}
```

---

# 23. Custom Constructors in Records

A record can have additional constructors, but they must ultimately delegate to the canonical constructor.

Example:

```java
public record Person(String name, int age) {

    public Person(String name) {
        this(name, 0);
    }
}
```

The additional constructor delegates to the canonical constructor:

```java
this(name, 0);
```

A constructor in a record cannot bypass the canonical construction process.

---

# 24. Records Can Have Methods

A record is still a class, so we can define methods inside it.

```java
public record Person(String name, int age) {

    public boolean isAdult() {
        return age >= 18;
    }
}
```

Usage:

```java
Person person = new Person("Sekhar", 27);

System.out.println(person.isAdult());
```

---

# 25. Records Can Implement Interfaces

Records can implement interfaces.

```java
public interface Printable {
    void print();
}
```

```java
public record Person(String name, int age)
        implements Printable {

    @Override
    public void print() {
        System.out.println(name + " - " + age);
    }
}
```

A record cannot extend an arbitrary class.

All records implicitly extend:

```java
java.lang.Record
```

Since Java classes support single inheritance, a record cannot extend another class.

---

# 26. Records Cannot Have Additional Instance Fields

A record cannot declare additional instance fields.

Invalid:

```java
public record Person(String name, int age) {

    private String address; // Compilation error
}
```

If the data needs to be part of the record's state, it must be declared as a record component:

```java
public record Person(
        String name,
        int age,
        String address
) {
}
```

However, records can have:

- Static fields
- Static methods
- Instance methods
- Constructors
- Nested types

Example:

```java
public record Person(String name, int age) {

    public static final String TYPE = "PERSON";

    public boolean isAdult() {
        return age >= 18;
    }
}
```

---

# 27. Quick Comparison

| Feature                       | Normal Class    | Sealed Class    | Record                                 |
| ----------------------------- | --------------- | --------------- | -------------------------------------- |
| Can have instance fields      | Yes             | Yes             | No additional instance fields          |
| Can extend another class      | Yes             | Yes             | No                                     |
| Can implement interfaces      | Yes             | Yes             | Yes                                    |
| Controls inheritance          | No              | Yes             | Inherently cannot be extended          |
| Automatic `equals()`          | No              | No              | Yes                                    |
| Automatic `hashCode()`        | No              | No              | Yes                                    |
| Automatic useful `toString()` | No              | No              | Yes                                    |
| Designed primarily for data   | Not necessarily | Not necessarily | Yes                                    |
| Mutable state possible        | Yes             | Yes             | Record components cannot be reassigned |

---

# Mistakes / Corrections From This Session

## 1. `var` is not a dynamically typed variable

Incorrect idea:

> `var` means the variable's type is determined dynamically.

Correct:

```java
var x = 10;
```

The compiler infers `int` at compile time. After compilation, `x` is still statically typed as `int`.

---

## 2. `var` cannot be used as a field because it is specifically local variable type inference

It is not because the memory must be stored in a particular place.

Incorrect reasoning:

> We cannot use `var` for an instance variable because it would be stored differently in memory.

Correct reasoning:

`var` is specifically a feature for **local variable type inference**. Java does not allow it for fields, method parameters, or return types.

---

## 3. Don't think of the memory model simply as "var goes to stack"

The important distinction is:

```java
var person = new Person();
```

`person` is a local reference variable, while the `Person` object is created on the heap in the conventional JVM memory model.

The exact JVM memory behavior can involve optimizations, so avoid memorizing "all local variables are always physically stored on the stack."

---

## 4. `var[] arr` is invalid, but `var arr = new int[10]` is valid

Correct:

```java
var arr = new int[10];
```

Incorrect:

```java
var[] arr = new int[10];
```

The compiler infers `int[]` from the initializer.

---

## 5. Records were not introduced in Java 2D

Records were introduced as a preview feature in **Java 14** and became a permanent feature in **Java 16**.

---

## 6. A record does not extend a class called `record`

A record implicitly extends:

```java
java.lang.Record
```

It cannot extend another class.

It **can** implement interfaces.

---

## 7. Record components are not simply "immutable objects"

The record components are final references/values, but this does not guarantee deep immutability.

For example:

```java
public record Team(List<String> members) {
}
```

The `members` reference cannot be reassigned, but the list itself may still be mutable.

---

## 8. `toString()` does not literally print the memory address

For a normal class that does not override `toString()`, `Object.toString()` produces a representation based on the class name and hash code.

It is better to say:

> The default `Object.toString()` produces a class-name/hash-code representation.

It should not be described as literally printing the object's memory address.

---

## 9. Records automatically provide `equals()`, `hashCode()`, and `toString()`

You don't normally need to manually implement these for a record.

For:

```java
public record Person(String name, int age) {
}
```

the record automatically provides appropriate implementations based on its components.

---

## 10. A record is not necessarily deeply immutable

Records prevent reassignment of their component fields, but if a component refers to a mutable object, that object can still be changed.

```java
public record Person(List<String> hobbies) {
}
```

The reference `hobbies` is final, but the list may be modified.

---

## 11. Sealed interface inheritance

If:

```java
sealed interface X permits Y
```

and `Y` directly extends `X`, then `Y` must be:

```java
sealed
```

or:

```java
non-sealed
```

It cannot be `final`, because interfaces cannot be declared `final`.

---

## 12. "Static arrays" are not a separate array category in Java

Java arrays have fixed size after creation:

```java
var arr = new int[10];
```

The size cannot be changed, but this is not what "static array" means in the `var` context.

The important rule is simply:

```java
var arr = new int[10];       // valid
var arr = new int[]{1, 2, 3}; // valid
var arr = {1, 2, 3};        // invalid
```

```

```
