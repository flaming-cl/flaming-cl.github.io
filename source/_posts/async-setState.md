---
title: Is setState asynchronous?
date: 2023-02-07 15:07:27
tags: React
---
Whether setState is synchronous or asynchronous has become a cliché these days.   

However, as React is constantly evolving, a frontend programmer's answer to this question can still demonstrate some deep understanding of the framework.

Today, let's gain a deeper understanding of some implementation mechanisms behind React.

## Ideas of this article 
Although `setState()` is not inherently asynchronous, React has designed it to behave like an asynchronous function for certain reasons. This article will:

- Summarize why `setState()` behaves like an asynchronous function.
- Explore the reasons why synchronous `setState()` can behave asynchronously.
- Clarify whether `setState()` behaves like a microtask or a macrotask.

## Why setState() act like an async function
(This section is a recap on Dan Abramov's response to the question posed in [RFClarification: why is setState asynchronous?](https://github.com/facebook/react/issues/11527#issuecomment-360199710))

In daily life, tasks that require immediate attention are important to us, while tasks we procrastinate on are often less critical. Similarly, not every state update triggered by `setState()` is of the highest priority.

This is where the first reason of asynchronously `setState` comes into play: priority scheduling.
> React could assign different priorities to setState() calls depending on where they’re coming from: an event handler, a network response, an animation, etc.


### Reason 1: ensures concurrent features
**Example**
Imagine you're editing a post on social media while getting a dozen of new notification messages.
If there are 100+ or even 500+ new messages, there may be no response to your keyboard input, as the browser is busy updating new notifications.

**For the above situation, what can you do to ensure smooth text input for users?**
A good practice is: giving low priority to message updates, as they are less important in this case. And when a message update encounters a high priority event, we will ask the former one to yield the main thread.

**Why asynchronous `setState()` benefits prioritized rendering (concurrent features) in React?**
This is because async-like `setState()` makes concurrent features possible, by delaying execution of low priority task and making room for high priority ones.

### Reason 2: avoid dirty data
Besides performance optimization, avoiding dirty data is another reason not to run `setState()` in a synchronous way. To understand this, we need to talk about the core value of React first.

**React In Theory**

For React, its most fundamental principle evolved from this formula:
> UI = f(state)

The concept behind this idea is straightforward: if you have the same input states for your application, it should always produce the same user interface (UI).

However, running `setState()` synchronously may violate React's pure-function-like update processes:

**It is hard to ensure consistency on synchronously updated states and their props.**
When a sync `setState()` is fired, its related props have to wait for reconciliation or flushSync to happen for corresponding updates. Such inconsistency can lead to unpredictable state updates in a React App.

## What makes sync setState() act asynchronously
Every time `renderRoot` or `setState` is triggered, React doesn't immediately start rendering in a React App. Instead, React will first schedule the updates, assigning different priority levels to each task and combining multiple tasks into one.

Because of this, synchronous `setState()` calls are collected in an `updateQueue`.  
The way `updateQueue` implemented gives `setState()` asynchronous behaviors: 
- `normal setState()` calls will be handled within a micro task queue
- `concurrent featured setState()` calls are handled in a macro task queue

**You might be wondering how to prove that `setState()` itself is synchronous?**
A classic example is: Prior to React 18, if you called `setState()` within `setTimeout()`, you would notice that it executes immediately. (In React 18, this issue has been addressed, and `setState()` calls made within `setTimeout()` are now asynchronous as well)


## Microtask or Macrotask
When updating, if React encounters a high-priority task, it will not execute the next low-priority task and schedule the highest-priority task to be executed.

**How does React implement this (let high priority tasks cut in line)?**
When it comes to the JavaScript event loop, a common method of interrupting is to let the newly added microtask interrupt the execution of existing macro tasks.
We also know that React update functions (setState) are often processed as microtasks.

**Take the concurrent feature as an example:**
To interrupt low-priority tasks, we mark the tasks in a concurrent feature API as low-priority and put them in a macro task queue.
In this way, when higher-priority microtask updates appear, they can jump ahead of the current low-priority macro tasks and be processed first.

## Reference
1. [RFClarification: why is setState asynchronous?](https://github.com/facebook/react/issues/11527#issuecomment-360199710)
2. [Automatic batching for fewer renders in React 18](https://github.com/reactwg/react-18/discussions/21)
