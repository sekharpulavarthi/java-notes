# Java Day 7 — Constructors, Static, Class Loading & Instantiation

## 1. Default Values of Instance Variables

Java gives default values to instance variables when an object is created.

Common examples:

```text
int       → 0
float     → 0.0
double    → 0.0
boolean   → false
char      → '\u0000'
String    → null
```

Example:

```java
class Student {
    int marks;
    String name;
}
```

```java
Student s = new Student();
```

Initially:

```text
s.marks → 0
s.name  → null
```

---

# 2. Constructor

A **constructor** is a special member of a class that is invoked when an object is created.

Example:

```java
class Student {

    Student() {
        System.out.println("Constructor called");
    }
}
```

Creating an object:

```java
Student s = new Student();
```

causes the constructor to execute.

---

## 3. Constructor Rules

A constructor:

- Has the **same name as the class**
- Has **no return type**, not even `void`
- Is invoked when an object is created
- Can have parameters
- Can be overloaded

Example:

```java
class Student {

    Student() {
    }

    Student(int marks) {
    }
}
```

`public` is **not mandatory** for a constructor.

Constructors can have access modifiers such as:

```java
public
private
protected
```

or no modifier.

---

# 4. Parameterized Constructor

A constructor that accepts parameters is called a **parameterized constructor**.

```java
class Student {

    int marks;
    String name;

    Student(int marks, String name) {
        this.marks = marks;
        this.name = name;
    }
}
```

Creating objects:

```java
Student s1 = new Student(90, "Sekhar");
Student s2 = new Student(80, "Rahul");
```

Each object receives its own values.

```text
s1 → marks = 90, name = "Sekhar"

s2 → marks = 80, name = "Rahul"
```

---

# 5. Default Constructor vs No-Argument Constructor

A **no-argument constructor** is a constructor that takes no parameters:

```java
Student() {
}
```

A **default constructor**, specifically, is the constructor that the **compiler automatically provides if you don't declare any constructor**.

Example:

```java
class Student {
    int marks;
}
```

The compiler provides a constructor equivalent in effect to:

```java
Student() {
}
```

### Important

If you create **any constructor yourself**, the compiler does not automatically provide this default constructor.

```java
class Student {

    Student(int marks) {
    }
}
```

Now:

```java
new Student();   // ❌
```

because no no-argument constructor was declared.

---

# 6. Constructor Overloading

A class can have multiple constructors as long as their parameter lists differ.

```java
class Student {

    Student() {
    }

    Student(int marks) {
    }

    Student(int marks, String name) {
    }
}
```

This is **constructor overloading**.

---

# 7. `static` Variables

An instance variable belongs to an object.

```java
class Student {
    int marks;
}
```

Each object has its own `marks`.

A `static` variable belongs to the **class**, so there is one shared variable for that class.

```java
class Student {
    static String school = "ABC School";
}
```

Conceptually:

```text
Student class
      ↓
   school
   (shared)
      ↑
 ┌────┴────┐
 ↓         ↓
s1        s2
```

Both objects can access the same static variable.

---

## 8. Accessing Static Variables

Prefer accessing a static member through the class name:

```java
Student.school
```

rather than:

```java
s1.school
```

Although Java permits access through an object reference, using the class name makes it clear that the variable belongs to the class.

---

# 9. Static vs Instance Variables

```java
class Student {

    int marks;              // instance variable
    static String school;   // static variable
}
```

Conceptually:

```text
Instance variable
→ one copy per object

Static variable
→ shared by the class
```

Example:

```java
Student s1 = new Student();
Student s2 = new Student();

s1.marks = 90;
s2.marks = 80;

Student.school = "ABC";
```

Result:

```text
s1.marks → 90
s2.marks → 80

Student.school → "ABC"
```

---

# 10. Static Methods

A static method belongs to the class.

```java
class Calculator {

    static int add(int a, int b) {
        return a + b;
    }
}
```

It can be called using the class name:

```java
int result = Calculator.add(10, 20);
```

No `Calculator` object is required to call this static method.

---

# 11. Instance / Non-Static Methods

A non-static method belongs to an object.

```java
class Calculator {

    int add(int a, int b) {
        return a + b;
    }
}
```

Create an object first:

```java
Calculator calculator = new Calculator();

int result = calculator.add(10, 20);
```

You cannot call it directly using:

```java
Calculator.add(10, 20);   // ❌
```

---

# 12. Static Method and Instance Members

A static method does not have a specific object associated with it.

Therefore, it cannot directly access instance variables:

```java
class Student {

    int marks;

    static void display() {
        System.out.println(marks);  // ❌
    }
}
```

Which object's `marks` should be used?

```text
Student
 ├── s1 → marks = 90
 └── s2 → marks = 80
```

There is no particular object associated with the static method call.

You can access the instance variable through an object:

```java
static void display(Student s) {
    System.out.println(s.marks);
}
```

---

# 13. Static Method Can Access Static Members

A static method can directly access static members:

```java
class Student {

    static String school = "ABC";

    static void display() {
        System.out.println(school);
    }
}
```

Because both belong to the class.

---

# 14. Static Block

A static block is used to execute code when the class is initialized.

```java
class Student {

    static {
        System.out.println("Static block");
    }
}
```

A static block is associated with the **class**, not with each object.

It is executed when the class is initialized, normally before the class's static members are used and before an instance is created as part of that initialization.

Therefore, it is generally executed **once per class initialization**, not once for every object.

---

# 15. Constructor vs Static Block

```java
class Student {

    static {
        System.out.println("Static block");
    }

    Student() {
        System.out.println("Constructor");
    }
}
```

When the class is initialized and then the first object is created, conceptually:

```text
Class initialization
       ↓
Static block
       ↓
Object creation
       ↓
Constructor
```

Creating another object:

```java
new Student();
```

does not execute the static block again.

The constructor executes for each object creation.

```text
First object:
Static block → Constructor

Second object:
               Constructor

Third object:
               Constructor
```

---

# 16. Class Loading

The JVM uses a **Class Loader** to load class information when needed.

Conceptually:

```text
.class file
    ↓
Class Loader
    ↓
Class loaded into JVM
    ↓
Class initialization
```

Class loading and object creation are **different operations**.

A class can be loaded without creating an object.

---

# 17. What Does "Instantiate" Mean?

**Instantiation means creating an instance (object) of a class.**

Example:

```java
Student s = new Student();
```

Here:

```java
new Student()
```

creates an instance of `Student`.

Therefore:

```text
Class
  ↓
Instantiation
  ↓
Object / Instance
```

### Simple memory trick

> **Instantiate = create an object from a class.**

---

# 18. Class Loading vs Instantiation

These are different:

### Class loading

JVM loads the class information.

```text
Student.class
     ↓
Class Loader
     ↓
Student class loaded
```

### Instantiation

An object is created:

```java
Student s = new Student();
```

```text
Student class
     ↓
new Student()
     ↓
Student object
```

A class can be loaded without instantiating an object.

---

# 19. `Class.forName()`

A class can be explicitly requested to be loaded using:

```java
Class.forName("com.example.Student");
```

`Class.forName()` returns a `Class` object representing the class.

It is historically/common in examples involving JDBC and dynamic class loading.

Modern Java applications don't normally need to use it for ordinary class usage.

---

# 20. `main()` and `static`

The Java entry point is:

```java
public static void main(String[] args)
```

`main()` is `static` because the JVM needs to invoke the entry point **without first creating an instance of the class**.

If `main()` were an instance method, an object would be required before it could be invoked.

```text
JVM
 ↓
static main()
 ↓
application starts
```

---

# Quick Recall

```text
Constructor
→ same name as class
→ no return type
→ runs during object creation
→ can be overloaded

Default constructor
→ compiler-provided only when no constructor is declared

Parameterized constructor
→ constructor with parameters

Instance variable
→ belongs to object
→ separate copy for each object

static variable
→ belongs to class
→ shared

Instance method
→ called through object

static method
→ called through class
→ no specific object

static block
→ executes during class initialization
→ not once per object

Class loading
→ JVM loads class information

Instantiation
→ creating an object/instance

Class loading ≠ object creation

main()
→ static so JVM can invoke it without creating an object
```
