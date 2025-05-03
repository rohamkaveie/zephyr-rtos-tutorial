# 🧠 1,What is a Thread?

- A thread is a small "program" running inside a bigger program (the application).
- In Zephyr RTOS, each thread can run independently and at the same time (even if your CPU is single-core, Zephyr switches between them super fast so it feels parallel).
- A thread has:
  - Code to run (function).
  - Its own memory (stack).
  - State (running, suspended, sleeping, etc.).
- In simple words:
>   A thread is like a worker in a factory, doing one job while other workers do other jobs. 

---
# 🏗️ 2. Thread Life Cycle in Zephyr

A thread can be in these states:

-    Ready: waiting to run.
-    Running: actually executing.
-    Blocked: waiting for something (like sleep time or a lock).
-    Suspended: manually paused.
-    Dead: terminated.

---

# 📚 3. How to Create a Thread in Zephyr

In Zephyr, you create a thread using either:
-    k_thread_create() → C API way.
-    K_THREAD_DEFINE() → Macro way (easy at compile-time).

Each thread needs:
-    A function (what the thread does).
-    A stack (its memory space).
-    A priority (higher = more important).


**<p style="font-size: 18px;">Example 1: Create with ```k_thread_create```</p>**

```c
#include <zephyr/kernel.h>

// Stack memory for the thread
K_THREAD_STACK_DEFINE(my_stack_area, 1024);

// Thread control block
struct k_thread my_thread_data;

// Thread entry function
void my_thread_fn(void *arg1, void *arg2, void *arg3)
{
    while (1) {
        printk("Hello from my thread!\n");
        k_sleep(K_SECONDS(1));
    }
}

void main(void)
{
    printk("Main thread started\n");

    k_tid_t my_tid = k_thread_create(&my_thread_data, my_stack_area,
                                     K_THREAD_STACK_SIZEOF(my_stack_area),
                                     my_thread_fn,
                                     NULL, NULL, NULL,
                                     5, 0, K_NO_WAIT);
}
```

✅ Explanation:
-    K_THREAD_STACK_DEFINE() creates a stack.
-    k_thread_create() creates the thread.
-    my_thread_fn runs forever and prints.

---

**<p style="font-size: 18px;">Example 2: Create with ```K_THREAD_DEFINE``` (easier!)</p>**

```c
#include <zephyr/kernel.h>

void my_thread_fn(void *arg1, void *arg2, void *arg3)
{
    while (1) {
        printk("Hello from macro thread!\n");
        k_sleep(K_SECONDS(1));
    }
}

// Define the thread
K_THREAD_DEFINE(my_tid, 1024, my_thread_fn, NULL, NULL, NULL, 5, 0, 0);
```

✅ Explanation:
- K_THREAD_DEFINE automatically:
  - Reserves stack memory,
  - Creates the thread,
  - Starts it at boot.

---

# 🏁 4. How to Start, Sleep, Suspend, Resume, Abort Threads

## 🛫 4.1 Starting a Thread

- ```k_thread_create()``` starts the thread immediately unless you use a delay (last parameter).

## ⏸️ 4.2 Suspending a Thread

```c
k_thread_suspend(my_tid);
```

➡️ The thread stops running until resumed.


## ▶️ 4.3 Resuming a Suspended Thread

Resume the paused thread:

```c
k_thread_resume(my_tid);
```

## 💀 4.4 Aborting (Killing) a Thread

Kill a thread completely:
```c
k_thread_abort(my_tid);
```

➡️ After aborting, the thread is dead and cannot resume.

# 🔄 5. Thread Context Switching

- Context switch means: "the CPU stops running one thread and starts running another."
- Zephyr automatically switches threads:
  - Based on priority (higher priority = runs first).
  - Or when a thread sleeps, waits, or blocks.

# ⚡ 6. Thread Priorities in Zephyr

- Threads have a priority number.
- Lower number = Higher priority.
  - Priority 0 = most important.
  - Priority 1 = less important.
- Example:
```c
// Higher priority
k_thread_create(&thread1, ... , 2, ...);

// Lower priority
k_thread_create(&thread2, ... , 5, ...);
```
Thread1 (priority 2) runs before Thread2.

# 🧰 7. Complete Example with Suspend, Resume, and Abort

```c 
#include <zephyr/kernel.h>

K_THREAD_STACK_DEFINE(my_stack_area, 1024);
struct k_thread my_thread_data;
k_tid_t my_tid;

void my_thread_fn(void *arg1, void *arg2, void *arg3)
{
    while (1) {
        printk("Thread is running!\n");
        k_sleep(K_SECONDS(1));
    }
}

void main(void)
{
    printk("Main started\n");

    my_tid = k_thread_create(&my_thread_data, my_stack_area,
                             K_THREAD_STACK_SIZEOF(my_stack_area),
                             my_thread_fn,
                             NULL, NULL, NULL,
                             5, 0, K_NO_WAIT);

    k_sleep(K_SECONDS(3));

    printk("Suspending thread\n");
    k_thread_suspend(my_tid);

    k_sleep(K_SECONDS(3));

    printk("Resuming thread\n");
    k_thread_resume(my_tid);

    k_sleep(K_SECONDS(3));

    printk("Aborting thread\n");
    k_thread_abort(my_tid);
}
```
✅ What happens here:
- Start the thread.
- After 3 seconds: suspend it.
- After another 3 seconds: resume it.
- After another 3 seconds: abort it (kill it).
  
# 📈 8. Summary Table

| Operation        | Function           |
| ------------- |:-------------:|
| Create Thread | k_thread_create() |
| Define at compile-time | K_THREAD_DEFINE() |
| Suspend | k_thread_suspend() |
| Resume | k_thread_resume() |
| Abort | k_thread_abort() |
| Sleep | k_sleep() inside thread |
| Yield (give CPU to others) | k_yield() |
| Change priority | k_thread_priority_set() |

---

# 🔄 9. Manual Thread Switching with ```k_yield()``` 

Normally, Zephyr automatically switches threads (based on priority, sleeping, blocking).
But if you want a thread to "voluntarily" give up the CPU to let others run even if it’s not sleeping, you use ```k_yield()```.

**<p style="font-size: 18px;">Example:</p>**

```c
#include <zephyr/kernel.h>

K_THREAD_STACK_DEFINE(thread1_stack, 1024);
K_THREAD_STACK_DEFINE(thread2_stack, 1024);

struct k_thread thread1_data;
struct k_thread thread2_data;

void thread1_fn(void *arg1, void *arg2, void *arg3)
{
    while (1) {
        printk("Thread 1 running\n");
        k_yield(); // Give other threads a chance
    }
}

void thread2_fn(void *arg1, void *arg2, void *arg3)
{
    while (1) {
        printk("Thread 2 running\n");
        k_yield(); // Give other threads a chance
    }
}

void main(void)
{
    printk("Main thread\n");

    k_thread_create(&thread1_data, thread1_stack, K_THREAD_STACK_SIZEOF(thread1_stack),
                    thread1_fn, NULL, NULL, NULL,
                    3, 0, K_NO_WAIT);

    k_thread_create(&thread2_data, thread2_stack, K_THREAD_STACK_SIZEOF(thread2_stack),
                    thread2_fn, NULL, NULL, NULL,
                    3, 0, K_NO_WAIT);
}
```

**<p style="font-size: 18px;">📢 Important:</p>**
> If you don't call k_yield() (or k_sleep()), and your thread is infinite, it will hog the CPU and others won’t get a turn!

---

# 🔗 10. Two Threads Communicating (Shared Variable Example)

Threads often need to share data.
Simple (but risky!) way is using a global variable.

**<p style="font-size: 18px;">Example: Counter shared between two threads</p>**

```c
#include <zephyr/kernel.h>

K_THREAD_STACK_DEFINE(producer_stack, 1024);
K_THREAD_STACK_DEFINE(consumer_stack, 1024);

struct k_thread producer_data;
struct k_thread consumer_data;

volatile int shared_counter = 0; // Shared data

void producer_fn(void *arg1, void *arg2, void *arg3)
{
    while (1) {
        shared_counter++;
        printk("Produced: %d\n", shared_counter);
        k_sleep(K_MSEC(500));
    }
}

void consumer_fn(void *arg1, void *arg2, void *arg3)
{
    while (1) {
        printk("Consumed: %d\n", shared_counter);
        k_sleep(K_MSEC(500));
    }
}

void main(void)
{
    k_thread_create(&producer_data, producer_stack, K_THREAD_STACK_SIZEOF(producer_stack),
                    producer_fn, NULL, NULL, NULL,
                    3, 0, K_NO_WAIT);

    k_thread_create(&consumer_data, consumer_stack, K_THREAD_STACK_SIZEOF(consumer_stack),
                    consumer_fn, NULL, NULL, NULL,
                    3, 0, K_NO_WAIT);
}
```

✅ What happens:
- producer_fn increases a counter every 500ms.
- consumer_fn reads it.

**<p style="font-size: 18px;">BUT ❗ THIS IS DANGEROUS!</p>**

Two threads accessing the same variable at the same time ➔ race conditions 🔥

You need synchronization to protect it properly.

---

# 🔒 11. Proper Thread Synchronization

In Zephyr, the main synchronization tools are:

| Tool | What it Does |
| - | - |
| Mutex | Protects shared data (only one thread can use at a time). |
| Semaphore | Signaling mechanism between threads. |
| Message Queue | Passing small messages safely between threads. |

## 11.1 Mutex Example (protect shared variable)

```c
#include <zephyr/kernel.h>

K_THREAD_STACK_DEFINE(producer_stack, 1024);
K_THREAD_STACK_DEFINE(consumer_stack, 1024);

struct k_thread producer_data;
struct k_thread consumer_data;

volatile int shared_counter = 0;
struct k_mutex counter_mutex;

void producer_fn(void *arg1, void *arg2, void *arg3)
{
    while (1) {
        k_mutex_lock(&counter_mutex, K_FOREVER);

        shared_counter++;
        printk("Produced (mutex): %d\n", shared_counter);

        k_mutex_unlock(&counter_mutex);

        k_sleep(K_MSEC(500));
    }
}

void consumer_fn(void *arg1, void *arg2, void *arg3)
{
    while (1) {
        k_mutex_lock(&counter_mutex, K_FOREVER);

        printk("Consumed (mutex): %d\n", shared_counter);

        k_mutex_unlock(&counter_mutex);

        k_sleep(K_MSEC(500));
    }
}

void main(void)
{
    k_mutex_init(&counter_mutex);

    k_thread_create(&producer_data, producer_stack, K_THREAD_STACK_SIZEOF(producer_stack),
                    producer_fn, NULL, NULL, NULL,
                    3, 0, K_NO_WAIT);

    k_thread_create(&consumer_data, consumer_stack, K_THREAD_STACK_SIZEOF(consumer_stack),
                    consumer_fn, NULL, NULL, NULL,
                    3, 0, K_NO_WAIT);
}

```

## 11.2 Semaphore Example (signal between threads)

Semaphore = a counter for signaling.

Imagine: "Producer tells Consumer that something is ready".

```c
#include <zephyr/kernel.h>

K_THREAD_STACK_DEFINE(producer_stack, 1024);
K_THREAD_STACK_DEFINE(consumer_stack, 1024);

struct k_thread producer_data;
struct k_thread consumer_data;

struct k_sem sem;

void producer_fn(void *arg1, void *arg2, void *arg3)
{
    while (1) {
        printk("Producing item...\n");
        k_sleep(K_MSEC(500));

        k_sem_give(&sem); // Signal that item is ready
    }
}

void consumer_fn(void *arg1, void *arg2, void *arg3)
{
    while (1) {
        k_sem_take(&sem, K_FOREVER); // Wait for signal
        printk("Consumed item!\n");
    }
}

void main(void)
{
    k_sem_init(&sem, 0, 1);

    k_thread_create(&producer_data, producer_stack, K_THREAD_STACK_SIZEOF(producer_stack),
                    producer_fn, NULL, NULL, NULL,
                    3, 0, K_NO_WAIT);

    k_thread_create(&consumer_data, consumer_stack, K_THREAD_STACK_SIZEOF(consumer_stack),
                    consumer_fn, NULL, NULL, NULL,
                    3, 0, K_NO_WAIT);
}
```

## 11.3 Message Queue (k_msgq) Between Threads

📖 What is a Message Queue?
-A Message Queue is a mailbox.
-One thread sends messages.
-Another thread receives them safely.
-Zephyr queues the messages automatically (FIFO — First In First Out).
✅ Good for passing data between threads without fighting over shared memory!

---

**<p style="font-size: 18px;">📜 Example: Using k_msgq</p>**

Imagine a producer sending numbers to a consumer.

```c
#include <zephyr/kernel.h>

#define MSGQ_MAX_MSGS 5
#define MSGQ_MSG_SIZE sizeof(int)

K_THREAD_STACK_DEFINE(producer_stack, 1024);
K_THREAD_STACK_DEFINE(consumer_stack, 1024);

struct k_thread producer_data;
struct k_thread consumer_data;

// Define the Message Queue
K_MSGQ_DEFINE(my_msgq, MSGQ_MSG_SIZE, MSGQ_MAX_MSGS, 4);

void producer_fn(void *arg1, void *arg2, void *arg3)
{
    int count = 0;

    while (1) {
        printk("Producer sending: %d\n", count);
        k_msgq_put(&my_msgq, &count, K_FOREVER);

        count++;
        k_sleep(K_MSEC(500));
    }
}

void consumer_fn(void *arg1, void *arg2, void *arg3)
{
    int received;

    while (1) {
        k_msgq_get(&my_msgq, &received, K_FOREVER);
        printk("Consumer received: %d\n", received);
    }
}

void main(void)
{
    printk("Main started\n");

    k_thread_create(&producer_data, producer_stack, K_THREAD_STACK_SIZEOF(producer_stack),
                    producer_fn, NULL, NULL, NULL,
                    3, 0, K_NO_WAIT);

    k_thread_create(&consumer_data, consumer_stack, K_THREAD_STACK_SIZEOF(consumer_stack),
                    consumer_fn, NULL, NULL, NULL,
                    3, 0, K_NO_WAIT);
}

```
✅ What's happening:
- producer_fn sends an integer to the message queue.
- consumer_fn waits (blocking) and reads it when available.
✅ ```K_FOREVER``` means wait forever if queue is full (producer) or empty (consumer).

✅ ```K_MSGQ_DEFINE()``` creates the queue at compile time:
- Each message = sizeof(int)
- Can store up to 5 messages
- 4-byte alignment (common for CPUs)

---

# 🎨 12. Thread State Diagram in Zephyr RTOS

```sql
    +---------------+
    |  Not Created   |
    +---------------+
           |
       (k_thread_create)
           ↓
    +---------------+
    |    Ready       |<-------------+
    +---------------+               |
           |                        | (k_thread_resume)
     (Scheduled)                    |
           ↓                        |
    +---------------+               |
    |   Running      |              |
    +---------------+               |
    |   |   |                       |
    |   |   | k_sleep()             |
    |   |   | wait for I/O          |
    |   ↓   ↓                       |
    | +-----------------+           |
    | |   Suspended      |          |
    | +-----------------+           |
    |        ↑                      |
    |        | (k_thread_suspend)   |
    +--------+----------------------+
             |
       (Blocked, Timeouts)
             ↓
    +---------------+
    |   Blocked     |
    +---------------+
             |
       (Event happens, time passes)
             ↓
    +---------------+
    |    Ready       |
    +---------------+
             |
         (k_thread_abort)
             ↓
    +---------------+
    |   Dead         |
    +---------------+

```

---

# 🔥 Quick Recap Now:

| Concept | Example |
| - | - |
| Voluntary yield | k_yield() |
| Sleep for time | k_sleep(K_MSEC(500)) |
| Suspend | k_thread_suspend(tid) |
| Resume | k_thread_resume(tid) |
| Abort | k_thread_abort(tid) |
| Share safely | k_mutex |
| Signal | k_sem |
| Pass messages | k_msgq |

---

# 🛑 1. Priority Inversion Problem

🧠 What is Priority Inversion?
Priority inversion happens when:
- A high-priority thread is waiting on a low-priority thread because of a shared resource (e.g., mutex lock).
✅ Example:
1. Low Priority Task (Low_Prio_Thread) locks a mutex.
2. High Priority Task (High_Prio_Thread) wants the mutex but must wait.
3. Medium Priority Task (Mid_Prio_Thread) runs normally and blocks the Low_Prio_Thread from releasing the mutex.
Result ➔ High_Prio_Thread is stuck, delayed, even though it's high priority!

**<p style="font-size: 18px;">⚙️ Priority Inversion Visual Example</p>**

```sql
Timeline:
------------------------------->
Low_Prio_Thread: [Lock]--------[Blocked][Unlock]
Mid_Prio_Thread:        [Running][Running][Running]
High_Prio_Thread:             [Wants Mutex][Blocked][Blocked]
```
Notice: High-priority thread is blocked indirectly because Medium is running!

---

**<p style="font-size: 18px;">🛠️ Solution: Priority Inheritance!</p>**

When Priority Inheritance is enabled:
- Low_Prio_Thread temporarily inherits the higher priority.
- It finishes faster and unlocks the mutex quicker.
✅ Zephyr automatically supports Priority Inheritance with its k_mutex implementation!
(You don't need to manually configure it — k_mutex does this for you!)

## 2.2 Wait for Multiple Events (Advanced - k_poll)

```c
struct k_poll_event my_events[2];

k_poll_event_init(&my_events[0], K_POLL_TYPE_SEM_AVAILABLE, K_POLL_MODE_NOTIFY_ONLY, &sem1);
k_poll_event_init(&my_events[1], K_POLL_TYPE_SEM_AVAILABLE, K_POLL_MODE_NOTIFY_ONLY, &sem2);

k_poll(my_events, 2, K_FOREVER); // Wait for any of sem1 or sem2

```

# 🚀 3. Advanced Concept: Real Producer-Consumer Using Both Semaphore + Message Queue

🔥 Full Example:

```c
#include <zephyr/kernel.h>

#define MSGQ_MAX_MSGS 5
#define MSGQ_MSG_SIZE sizeof(int)

K_THREAD_STACK_DEFINE(producer_stack, 1024);
K_THREAD_STACK_DEFINE(consumer_stack, 1024);

struct k_thread producer_data;
struct k_thread consumer_data;

// Message Queue
K_MSGQ_DEFINE(my_msgq, MSGQ_MSG_SIZE, MSGQ_MAX_MSGS, 4);

// Semaphore
struct k_sem data_ready_sem;

void producer_fn(void *arg1, void *arg2, void *arg3)
{
    int count = 0;

    while (1) {
        printk("Producer created item: %d\n", count);
        k_msgq_put(&my_msgq, &count, K_FOREVER);
        k_sem_give(&data_ready_sem); // Signal data is ready
        count++;
        k_sleep(K_MSEC(500));
    }
}

void consumer_fn(void *arg1, void *arg2, void *arg3)
{
    int received;

    while (1) {
        k_sem_take(&data_ready_sem, K_FOREVER); // Wait for producer signal
        k_msgq_get(&my_msgq, &received, K_FOREVER);
        printk("Consumer processed item: %d\n", received);
    }
}

void main(void)
{
    k_sem_init(&data_ready_sem, 0, 1);

    k_thread_create(&producer_data, producer_stack, K_THREAD_STACK_SIZEOF(producer_stack),
                    producer_fn, NULL, NULL, NULL,
                    3, 0, K_NO_WAIT);

    k_thread_create(&consumer_data, consumer_stack, K_THREAD_STACK_SIZEOF(consumer_stack),
                    consumer_fn, NULL, NULL, NULL,
                    3, 0, K_NO_WAIT);
}

```
















    
    

