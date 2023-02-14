---
title: Is setState asynchronous?
date: 2023-02-07 15:07:27
tags: React
---
## Ideas of this article 
SetState itself is **not an asynchronous** function, but for some reasons React makes setState **act like an async** function.

This article will:
1. summarize why setState() act like an async function.
2. discuss what makes synchronous setState() to act asynchronously
3. provide an example to tell why async or sync setState matters  

## Why setState() act like an async function
We expect synchronous calls to be executed immediately, while asynchronous calls to be executed after a while.

**In daily life, what kind of things you will do immediately, and what you will do later?**

Obviously, things that are done immediately usually have higher priority, and things delayed are usually less important. Similarly, what we trigger with setState is not always with top priority.

Here comes the first reason to make setState act in an asynchronous way: **priority scheduling**. 

> React could assign different priorities to setState() calls depending on where they’re coming from: an event handler, a network response, an animation, etc.  
> -- Dan Abramov


### Reason 1: ensures concurrent features
**Example**
Assume you are receiving a bunch of notification messages, while editing a post on social media.

**What happens if the notification updates were immediately executed?** 

You may not be annoyed by this when there are only a few messages. But if there are 100+ or even 1000+ new messages, your keyboard input can be very sluggish, as the browser is busy updating new notifications.

**For the above situation, what can you do to ensure smooth text input for users?**

A good practice is: giving low priority to message updates which are not important in this case. And when a message update encounters a high priority event, we will ask the former one to yield the main thread.

**Why asynchronous setState() benefits prioritized rendering (concurrent features) in React?**
Asynchronous-like setState() makes concurrent features possible. 

This is because async-like setState() makes it easier to delay execution of low priority task and make room for high priority ones.

To further understand concurrent features, you can go to [this post](https://flaming-cl.github.io/post/simple-ideas-about-React-Concurrent-mode)

### Reason 2: avoid dirty data
Besides performance optimization, avoiding dirty data is another reason for us to avoid running setState() in an synchronous way.

To understand this, we need to talk about the core value of React first.

**React In Theory**

For React, its most fundamental principle evolved from this formula:
> UI = f(state)

The idea behind it is simple: **same input states result in same UI**.

However, based on Dan Abramov’s explanation, running setState() synchronously may violate React's pure-function-like update processes.

**Why running synchronous setState() may violate the core value of React?**

Because it is hard to ensure consistency on synchronously updated states and their props.

States are often not isolated but come with props. To ensure consistent data flow, once you immediately update a state, you also need to update its related props in time.

> However, even though you may immediately update a state, props only update after the reconciliation and flushing.

For example, although a parent component updates a state immediately, the state related props received by children have to wait until reconciliation or flushing finished.

**Such inconsistency can lead to dirty data.**

## What makes sync setState() act asynchronously

