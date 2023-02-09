---
title: Why is setState asynchronous?
date: 2023-02-07 15:07:27
tags: React
---
This is a reflection on Dan Abramov's answer in the RFClarification: why is setState asynchronous.

## What
Before figuring out why synchronous or asynchronous, I want to explain what they mean first.
### What is asynchronous and what is synchronous?
To put it simply, we expect synchronous calls to be executed immediately, while asynchronous calls will be executed after a while.

### In daily life, what kind of things would you do immediately, and what would you do later?
Obviously, things that are done immediately usually have higher priority, and things that are delayed are usually less important.

Similarly, what you trigger with setState is not always the most urgent, or may even be <abbr>insignificant</abbr>. So why update it immediately in a synchronous way?

## Reason 1: ensures concurrent features
### Example
Assume you are editing a post on social media. Meanwhile, you are receiving a bunch of notification messages.
### What if the notification updates were immediately executed? 
You may not be annoyed by this, when there are only a few messages. But if there are 100+ or even 1000+ new messages, your keyboard input can become sluggish, as the browser is busy with immediately processed (synchronous) new messages.

### For the above situation, what can you do to ensure smooth text input for users?
The solution is simple: give low priority to the message updates. When a message update encounters a high priority event, we will ask the former one to yield the main thread.
> React could assign different priorities to setState() calls depending on where they’re coming from: an event handler, a network response, an animation, etc.

### What is relationship between asynchronous setState calls and concurrent features?
It is the asynchronous feature of setState that makes concurrent features (rendering with priorities) possible. If all the setState calls are executed immediately, it is not easy for us to delay its execution and make room for high priority tasks.

To future understand benefits of concurrent features, you can go to [this post](https://flaming-cl.github.io/bits-refinery/bits-refinery/2023/02/06/simple-ideas-about-React-Concurrent-mode/)

## Reason 2: avoid dirty data
Besides performance concern, dirty data is another potential cost of synchronous setState.
To understand this, we need to talk about the core value of React first.

### React In Theory
For React, its most fundamental principle evolved from this formula:
> UI = f(state)

The idea behind it is simple: same state data leads to same UI.
However, based on Dan’s explanation, immediately executing setState may violate React's pure function-like update process:

### Why synchronous setState may violate the core theory of React?
It is hard to ensure consistency on synchronously updated states and their props.

To ensure consistent data flow, once you immediately update a state, you also need to update its related props in time.

Unfortunately, there is no quick solution to such requirement.
> You may immediately update a state, but props only update after the reconciliation and flushing.

So it is possible that a parent component updates one of its states immediately, but the related props received by children component have not changed yet. 
Such inconsistency can lead to dirty data.

