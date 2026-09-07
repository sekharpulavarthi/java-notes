# Java — Multithreading

## 1. What Is a Thread?

A thread is an **independent path of execution within a process**.

A program can have multiple threads performing different tasks concurrently.

Example:

```text
Application
│
├── Thread 1 → download data
├── Thread 2 → process data
└── Thread 3 → handle user interaction
```

Multiple threads can share resources belonging to the same process.

---

# 2. Concurrency vs Parallelism

### Concurrency

Multiple tasks make progress during overlapping periods.

The CPU may switch between threads:

```text
Thread A → Thread B → Thread A → Thread C → ...
```

### Parallelism

Multiple tasks actually execute simultaneously, typically on different CPU cores.

So don't always say:

> "Multiple threads run at exactly the same time."

Better:

> **Multithreading allows multiple threads to execute concurrently, and they may execute in parallel on multiple CPU cores.**

---

# 3. Creating a Thread Using `Thread`

One approach is extending `Thread`:

```java
class A extends Thread {

    public void run() {
        for (int i = 1; i <= 5; i++) {
            System.out.println("Hi");
        }
    }
}
```

Then:

```java
A obj1 = new A();

obj1.start();
```

Calling:

```java
obj1.start();
```

asks Java to start a new thread.

The new thread eventually executes:

```java
run()
```

---

# 4. `start()` vs `run()`

This is extremely important.

```java
obj.start();
```

→ starts a new thread and causes `run()` to execute on that thread.

But:

```java
obj.run();
```

→ is simply a normal method call.

It does **not** create a new thread.

Remember:

```text
start()
→ new thread execution

run()
→ normal method if called directly
```

---

# 5. Multiple Threads

```java
class A extends Thread {

    public void run() {
        for (int i = 1; i <= 5; i++)
            System.out.println("Hi");
    }
}

class B extends Thread {

    public void run() {
        for (int i = 1; i <= 5; i++)
            System.out.println("Hello");
    }
}
```

```java
A obj1 = new A();
B obj2 = new B();

obj1.start();
obj2.start();
```

Possible output:

```text
Hi
Hello
Hi
Hi
Hello
Hello
...
```

The exact order is **not guaranteed**.

The thread scheduler decides when threads get execution time.

---

# 6. Does Every Class Need `run()`?

No.

Only the class/object intended to provide the thread's task needs to implement/override `run()`.

For example:

```java
class A extends Thread {
    public void run() {
        // task
    }
}
```

A normal unrelated class:

```java
class Z {
    public void show() {
        // normal method
    }
}
```

doesn't need `run()`.

Also, a class can have many other methods:

```java
class A extends Thread {

    public void run() {
        // thread task
    }

    public void show() {
        // normal method
    }

    public void calculate() {
        // normal method
    }
}
```

`run()` is the method that defines the task executed by the thread when `start()` is used.

---

# 7. Thread Priority

Java provides priorities:

```java
Thread.MIN_PRIORITY
Thread.NORM_PRIORITY
Thread.MAX_PRIORITY
```

Typically:

```text
MIN_PRIORITY    = 1
NORM_PRIORITY   = 5
MAX_PRIORITY    = 10
```

Example:

```java
obj2.setPriority(Thread.MAX_PRIORITY);
```

Priority is a **scheduling hint**, not a guarantee.

A higher-priority thread does not mean:

> "This thread will definitely execute first."

The actual scheduling behavior depends on the JVM/OS/thread scheduler.

---

# 8. `Thread.sleep()`

```java
Thread.sleep(10);
```

pauses the **currently executing thread** for approximately the specified duration.

It can throw:

```text
InterruptedException
```

so it commonly needs to be handled or declared.

Example:

```java
try {
    Thread.sleep(10);
}
catch (InterruptedException e) {
    e.printStackTrace();
}
```

Important:

> `sleep()` does not guarantee that another specific thread will execute next.

It only makes the current thread unavailable for that sleep period.

Therefore, even with `sleep()`, you cannot guarantee:

```text
Hi
Hello
Hi
Hello
```

every time.

---

# 9. Creating a Thread Using `Runnable`

`Runnable` is an **interface**, not a class.

It has one abstract method:

```java
void run();
```

Therefore, it is a **functional interface** and can be used with a lambda.

Example:

```java
class A implements Runnable {

    public void run() {
        System.out.println("Hi");
    }
}
```

Then:

```java
Runnable obj = new A();

Thread t1 = new Thread(obj);

t1.start();
```

Here there are two different concepts:

```text
Runnable object
→ defines WHAT the task is

Thread object
→ provides the thread that executes the task
```

---

# 10. Why `Runnable` Is Useful

Java doesn't support multiple inheritance of classes.

Suppose:

```java
class A extends SomeOtherClass {
}
```

Now A cannot also do:

```java
class A extends Thread { } // impossible
```

because a class can extend only one class.

With `Runnable`:

```java
class A extends SomeOtherClass implements Runnable {

    public void run() {
        // task
    }
}
```

So we can inherit from another class while still defining a runnable task.

This is one important reason `Runnable` is generally preferable to extending `Thread` when you only need to define a task.

---

# 11. Anonymous Class + Runnable

Because `Runnable` is a functional interface, we can create its implementation with an anonymous class:

```java
Runnable obj = new Runnable() {

    public void run() {
        System.out.println("Hello");
    }
};
```

---

# 12. Lambda + Runnable

Because `Runnable` has one abstract method, we can simplify it:

```java
Runnable obj = () -> {
    System.out.println("Hello");
};
```

Then:

```java
Thread t = new Thread(obj);
t.start();
```

Or directly:

```java
Thread t = new Thread(() -> {
    System.out.println("Hello");
});

t.start();
```

---

# 13. Stack/Heap Mental Model

For:

```java
Runnable obj1 = () -> {
    System.out.println("Hi");
};

Thread t1 = new Thread(obj1);

t1.start();
```

Simplified:

```text
Stack
────────────────
obj1 ──────────────┐
t1 ────────────────┼──→ heap objects
                   │
Heap               │
────────────────   │
Runnable object ←──┘
Thread object  ←───┘
```

The `Thread` object represents/configures the thread of execution, while the `Runnable` provides the task to execute.

The actual execution uses JVM/OS thread scheduling.

---

# Quick Thread Recall

```text
Thread
→ independent path of execution within a process

Multithreading
→ multiple threads executing concurrently

start()
→ starts a new thread

run()
→ contains the task
→ direct run() call does NOT create a new thread

Thread.sleep()
→ pauses current thread temporarily
→ InterruptedException

Thread priority
→ scheduling hint, not guarantee

Runnable
→ interface
→ has run()
→ functional interface

Runnable
→ defines the task

Thread
→ executes the task

Lambda
→ possible because Runnable is a functional interface
```
