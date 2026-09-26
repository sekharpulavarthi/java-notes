# Java OOP — Classes, Objects & Methods

## 1. Object-Oriented Programming

Java is an **object-oriented programming language**.

An object can be understood in terms of:

- **State** → what it is / what data it has
- **Behavior** → what it does / what actions it can perform

Example:

```text
Car
├── State
│   ├── color
│   ├── speed
│   └── model
│
└── Behavior
    ├── start()
    ├── accelerate()
    └── brake()
```

---

## 2. Class

A **class is a blueprint/template** used to define the structure and behavior of objects.

It can contain:

- **Fields/variables** → represent state/data
- **Methods** → represent behavior/actions

Example:

```java
class Car {
    String color;
    int speed;

    void start() {
        System.out.println("Car started");
    }

    void brake() {
        System.out.println("Car stopped");
    }
}
```

The class itself is the **definition**. It is not the actual car object.

---

## 3. Object

An **object is an instance of a class**.

If `Car` is the blueprint:

```text
Class
  ↓
Car
```

Objects can be created from it:

```text
Car class
   ↓
   ├── Car object 1
   ├── Car object 2
   └── Car object 3
```

Each object can have its own state.

---

## 4. Creating an Object

Use the `new` keyword to create an object.

```java
Car car = new Car();
```

Breakdown:

```text
Car car = new Car();
│   │       │
│   │       └── Creates a Car object
│   └────────── Reference variable
└────────────── Type
```

The expression:

```java
new Car()
```

creates a new `Car` object and produces a reference to that object.

---

## 5. Using an Object

Once an object is created, we can use its fields and methods through the reference:

```java
Car car = new Car();

car.color = "Red";
car.speed = 100;

car.start();
car.brake();
```

Here:

```text
car.color  → state
car.speed  → state

car.start() → behavior
car.brake() → behavior
```

---

## 6. Class vs Object

| Class                                    | Object                                   |
| ---------------------------------------- | ---------------------------------------- |
| Blueprint/template                       | Instance of the class                    |
| Defines structure and behavior           | Represents an actual instance            |
| Describes what an object will contain/do | Contains its own state                   |
| Example: `Car`                           | Example: `car` created using `new Car()` |

Memory trick:

```text
Class  → Blueprint
Object → Actual instance
```

---

## 7. Multiple Classes in a Program

A Java application can contain many classes.

For example, a car-related application could have:

```text
Car
├── Engine
├── Wheel
├── Seat
└── Dashboard
```

Each class can define its own:

- State
- Behavior

Example:

```java
class Wheel {
    int size;

    void rotate() {
        System.out.println("Wheel rotating");
    }
}
```

A larger application can combine objects created from different classes.

---

## 8. Methods

A **method defines behavior/action** that an object or class can perform.

Example:

```java
void start() {
    System.out.println("Starting...");
}
```

General structure:

```java
accessModifier returnType methodName(parameters) {
    // method body
}
```

Example:

```java
public void start() {
    System.out.println("Starting...");
}
```

### Common parts

- `public` → access modifier
- `void` → method does not return a value
- `start` → method name
- `()` → parameter list

A method can also return a value:

```java
public int getSpeed() {
    return speed;
}
```

---

## 9. Access Modifier — `public`

`public` means the member can generally be accessed from anywhere the class itself is accessible.

Example:

```java
public void start() {
}
```

Access modifiers will be covered in more detail later.

---

## 10. Main Method

The `main()` method is the **entry point for a normally launched Java application**.

```java
public static void main(String[] args) {
    // execution starts here
}
```

The JVM invokes this method to start the application.

---

## 11. OOP Mental Model

```text
Problem / Requirement
        ↓
      Class
   (blueprint)
        ↓
      new
        ↓
     Object
        ↓
 State + Behavior
```

Example:

```text
        Car class
       /         \
      /           \
    state       behavior
   color         start()
   speed         brake()
      \           /
       \         /
        Car object
```

---

## 12. Simple Example

```java
class Calculator {

    int add(int a, int b) {
        return a + b;
    }

    int multiply(int a, int b) {
        return a * b;
    }
}

class Main {
    public static void main(String[] args) {

        Calculator calculator = new Calculator();

        int sum = calculator.add(10, 20);
        int result = calculator.multiply(5, 4);

        System.out.println(sum);
        System.out.println(result);
    }
}
```

Here:

```text
Calculator
    ↓
    Class / blueprint
    ↓
new Calculator()
    ↓
Calculator object
    ↓
calculator
    ↓
add() / multiply()
```

The `Calculator` class defines the behavior, while the created `Calculator` object is used to access that behavior.
