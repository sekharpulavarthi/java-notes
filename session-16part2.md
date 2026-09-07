# Java Multithreading — Thread Lifecycle & Synchronization

## 1. `join()` vs `synchronized`

These solve **two different problems**.

### `synchronized` → protects shared data

Example:

```java
class Counter {
    int count;

    public synchronized void increment() {
        count++;
    }
}
```

If two threads call `increment()` on the **same Counter object**, only one thread can execute the synchronized method at a time.

```text
Thread 1 → gets lock → increment → releases lock
Thread 2 → waits       → gets lock → increment → releases lock
```

Without `synchronized`, `count++` can cause a **race condition** because it involves:

```text
read count
    ↓
add 1
    ↓
write count
```

Two threads can read the same value before either writes the updated value.

### `join()` → makes one thread wait for another

```java
t1.start();
t2.start();

t1.join();
t2.join();

System.out.println(c.count);
```

`join()` means:

> "The current thread should wait until this thread finishes."

Here, the **main thread** waits for `t1` and `t2` before printing `count`.

```text
main
 │
 ├── start t1
 ├── start t2
 │
 ├── join t1 → WAIT
 │             ↓
 │          t1 finishes
 │
 ├── join t2 → WAIT
 │             ↓
 │          t2 finishes
 │
 └── print count
```

### Remember

```text
synchronized → protects shared data

join() → waits for thread completion
```

They are **not replacements for each other**.

---

# 2. Thread Lifecycle

Java officially defines **6 thread states**:

```text
NEW
RUNNABLE
BLOCKED
WAITING
TIMED_WAITING
TERMINATED
```

> "RUNNING" is commonly shown in simplified diagrams, but Java's official `Thread.State` does not have a separate RUNNING state. A thread that is actually executing is represented as `RUNNABLE`.

---

## 3. NEW

Thread object is created, but `start()` has not been called.

```java
Thread t1 = new Thread(obj1);
```

```text
NEW
```

The thread exists, but execution hasn't started.

### Transition

```text
new Thread()
     ↓
   NEW
```

---

# 4. RUNNABLE

Calling:

```java
t1.start();
```

makes the thread eligible to execute.

```text
NEW
 ↓
start()
 ↓
RUNNABLE
```

The JVM/OS scheduler decides when the thread actually gets CPU time.

Example:

```java
Thread t1 = new Thread(obj1);
t1.start();
```

Important:

```java
t1.run();
```

is **not** the same as:

```java
t1.start();
```

`start()` creates/starts a new thread of execution.

Calling `run()` directly is simply a normal method call on the current thread.

---

# 5. BLOCKED

A thread becomes `BLOCKED` when it is trying to acquire a monitor lock that another thread currently holds.

Example:

```java
class Counter {
    synchronized void increment() {
        count++;
    }
}
```

Suppose:

```text
Thread 1 → enters increment()
           gets Counter's lock

Thread 2 → tries to enter increment()
           ↓
         BLOCKED
```

When Thread 1 releases the lock, Thread 2 can become `RUNNABLE` again.

```text
RUNNABLE
   ↓
tries synchronized method
   ↓
lock unavailable
   ↓
BLOCKED
   ↓
lock becomes available
   ↓
RUNNABLE
```

### Quick recall

**BLOCKED = waiting for a lock.**

---

# 6. WAITING

A thread enters `WAITING` when it waits indefinitely for another thread/action.

Common examples:

```java
wait();
```

and:

```java
t1.join();
```

Example:

```java
t1.join();
```

If the main thread calls this:

```text
main
 ↓
t1.join()
 ↓
WAITING
 ↓
t1 finishes
 ↓
RUNNABLE
 ↓
main continues
```

Another example:

```java
obj.wait();
```

A thread can be awakened using:

```java
obj.notify();
```

or:

```java
obj.notifyAll();
```

`notify()` makes a waiting thread eligible to continue; it does **not guarantee that the thread runs immediately**.

### Quick recall

**WAITING = waiting indefinitely for something to happen.**

---

# 7. TIMED_WAITING

A thread enters `TIMED_WAITING` when it waits for a specified amount of time.

Examples:

```java
Thread.sleep(2000);
```

```java
t1.join(2000);
```

```java
obj.wait(2000);
```

Example:

```text
RUNNABLE
   ↓
sleep(2000)
   ↓
TIMED_WAITING
   ↓
approximately 2 seconds
   ↓
RUNNABLE
```

### Important difference

```java
sleep()
```

does **not release a lock** that the thread currently holds.

`wait()` releases the object's monitor while waiting.

### Quick recall

**TIMED_WAITING = waiting with a time limit.**

---

# 8. TERMINATED

When the thread's `run()` method finishes, the thread enters:

```text
TERMINATED
```

Example:

```java
Runnable task = () -> {
    System.out.println("Hello");
};
```

After the task completes:

```text
RUNNABLE
   ↓
run() finishes
   ↓
TERMINATED
```

A terminated Thread object cannot be started again.

```java
t1.start();
t1.start();   // IllegalThreadStateException
```

---

# 9. Complete Thread Lifecycle

```text
                 new Thread()
                      │
                      ↓
                    NEW
                      │
                   start()
                      │
                      ↓
                  RUNNABLE
                 ↙    ↓     ↘
                /     │       \
               /      │        \
        lock busy    wait()    sleep()
           ↓           ↓          ↓
        BLOCKED     WAITING   TIMED_WAITING
           │           │          │
           └───────────┴──────────┘
                       ↓
                   RUNNABLE
                       │
                 run() finishes
                       ↓
                  TERMINATED
```

---

# 10. Important Transitions

| Action            | State / Effect                              |
| ----------------- | ------------------------------------------- |
| `new Thread()`    | `NEW`                                       |
| `start()`         | `NEW → RUNNABLE`                            |
| Lock unavailable  | `RUNNABLE → BLOCKED`                        |
| `wait()`          | `WAITING`                                   |
| `join()`          | `WAITING`                                   |
| `sleep()`         | `TIMED_WAITING`                             |
| `join(timeout)`   | `TIMED_WAITING`                             |
| `wait(timeout)`   | `TIMED_WAITING`                             |
| `run()` completes | `TERMINATED`                                |
| `notify()`        | Waiting thread becomes eligible to continue |
| `stop()`          | Deprecated; don't use                       |

---

# 11. Counter Example — Complete Mental Model

```java
Counter c = new Counter();

Thread t1 = new Thread(obj1);
Thread t2 = new Thread(obj2);

t1.start();
t2.start();

t1.join();
t2.join();

System.out.println(c.count);
```

### Objects involved

Only **one Counter object**:

```text
                 Counter c
              ┌─────────────┐
              │ count       │
              └─────────────┘
                 ↑       ↑
                 │       │
              Thread 1  Thread 2
```

Both threads access the **same `count`**.

### `synchronized`

```java
public synchronized void increment()
```

protects the shared `count` from race conditions.

### `join()`

```java
t1.join();
t2.join();
```

makes `main` wait until both threads finish.

Therefore:

```text
synchronized → correct shared-data access

join() → main waits for completion
```

---

# Quick Interview Recall

### Thread states

```text
NEW
RUNNABLE
BLOCKED
WAITING
TIMED_WAITING
TERMINATED
```

### Remember the difference

```text
BLOCKED
→ waiting for a lock

WAITING
→ waiting indefinitely

TIMED_WAITING
→ waiting for a specified time

TERMINATED
→ thread has finished
```

### Most important methods

```text
start()  → starts a new thread
run()    → contains the task
join()   → wait for another thread to finish
sleep()  → pause current thread for a time
wait()   → wait and release monitor
notify() → wake a waiting thread
```

### Most important distinction

```text
synchronized
    ↓
Thread safety / protects shared data

join()
    ↓
Thread coordination / waiting for completion
```
