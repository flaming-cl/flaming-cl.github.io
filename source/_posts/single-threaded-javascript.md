---
title: Single-threaded JavaScript
date: 2023-02-08 14:33:14
tags: JavaScript
---
This article will illustrate synchronous/asynchronous and micro/macro tasks.
### Synchronous and Asynchronous Tasks
In JavaScript, we have synchronous and asynchronous tasks.

**Synchronous Tasks**
Synchronous tasks are processed immediately, and they can block JavaScript execution until the running task is completed.

**Asynchronous Tasks**
Asynchronous tasks, such as I/O or network requests, often require interaction with threads other than the JS engine main thread to obtain data for incoming execution.

As the main thread is not responsible for I/O or network jobs, it really doesn't need to wait for completion of long-running asynchronous tasks. To improve efficiency, JavaScript asks asynchronous tasks to yield the main thread for the next task, and will execute on ready asynchronous callbacks later.

---

#### Illustration for Asynchronous Tasks
![queuing](https://user-images.githubusercontent.com/51183663/219988659-2f282ff1-2788-49e5-adb1-dae052604aa5.png)
To better understand asynchronous tasks, let's look at an example of working as a counter staff at McDonald's. Here is a typical scenario of a counter staff's job:
1. have customers **waiting in line** before serving them
2. **serve** customer A to order food at the counter
3. tell customer A to **wait for food to be cooked**, and start serving customer B
4. when customer A's meal is ready, notify A to be back, pass meals to A and **complete service**

Let's take a look at how working as a counter staff at McDonald's is similar to task processing in JavaScript:
- **JavaScript is single-threaded, handling one task at a time.** 
The counter staff only serves one customer at a time.
- **JavaScript passes asynchronous tasks to web APIs, let callbacks of async calls to be executed later, and begin processing the next task.**
Food ordered by customer A is not cooked yet. The counter staff asks the kitchen to prepare the meal, tells customer A to go back later, and start serving customer B.
- **JavaScript keeps on ready asynchronous tasks waiting in queue before execution.**
The way customers wait in queue is quite similar to how on ready asynchronous tasks wait in the task queue, as they all follow a first in first out manner.

#### Illustration for Synchronous Tasks
**In JavaScript, synchronous tasks are executed immediately and before the execution of async tasks.**

The counter staff’s analogy can also be used to explain synchronous tasks, as if food ordered by customer B is in the holding cabinet and ready for picking up.   
In this case, the counter staff can directly pass the meal to and finish service for customer B, while customer A is still waiting for the meal to be ready.

---


### Inside asynchronous tasks: Microtask and Macrotask
To illustrate this, let's modify our previous story a bit. Now the McDonald's is inside a train station. It is common that some customers have their train leaving soon and ask other customers if they can cut in line to place their order soon.

**This scenario emphasizes the need for having both microtasks and macrotasks:** we want to make it possible for high priority tasks to cut in line and be executed ahead of less prioritized ones.

As a result, microtasks are designed to be prioritized and executed earlier, while macrotasks are less urgent and can be deferred. In other words, microtasks will be executed before macrotasks in JavaScript.

So far, we can infer the execution order of the three types of tasks is: 
> synchronous tasks -> microtasks -> macrotasks.

