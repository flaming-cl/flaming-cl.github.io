---
title: Questions About React concurrent features
date: 2023-02-06 12:21:10
tags: React
---

<summary>Do React concurrent features mean multitasking?</summary>
No. React concurrent features are not about multitasking.
This is because the JavaScript engine is single-threaded (can only do one thing at one time).
So there is no magic for React to simultaneously handle user events while rendering a large list.

### If so, what does React do to avoid stutters when running CPU heavy tasks?
In fact, React tries to avoid CPU bottlenecks by running one task, stopping it, running another, and so forth.
Sounds like a micro operating system, right?
But concurrent features will not allow tasks to freely compete for sources, instead they will label some tasks with low priority. This means these low priority tasks will yield the main thread to more urgent tasks to ensure performance.

This is quite similar to how we answer a second phone call:

> Assume there comes a food delivery call, while we are calling a friend for a casual chat.
Since a food delivery call is more urgent than a casual chat, we hold the chat and answer the delivery call.

![calls](https://user-images.githubusercontent.com/51183663/217071662-a89bd48e-beb5-4a82-ab62-7f36f7ab15fe.png)

React does the same thing. It can pause a less prioritized rendering task and yield to more urgent tasks, such as keyboard events which need immediate responds.

### Can you give some examples about React concurrent features?
Concurrent features: startTransition(), useTransition(), useDeferredValue()
By using the above APIs, you are telling React to give lower priority to things you put in them. 

1）useDeferredValue(value)
- example: input value of a search bar
- result: updates of the input value will be deferred, just like what "debounce" does

2）startTransition(callback)
- example: a function to update a large list
- result: When updating the large list, if there comes a keyboard event, React will pause the ongoing update and turn to the user event.

### What enables React to run the concurrent features?
Time slicing.
After V16, React started to embrace time slicing to ensure an interruptible render phase. 
This means React can insert urgent browser tasks among small tasks from the render phase to avoid stutter.

![217070500-dc1bfbcf-31a5-4158-8d19-7870dbf99071](https://user-images.githubusercontent.com/51183663/217111897-7dacf96a-a574-410b-865b-c5e8ce3c3f5a.png)
To understand more about time slicing, go to [this post](https://flaming-cl.github.io/bits-refinery/bits-refinery/2022/09/28/time-slicing-react/)

## References
[1] [Visualization of async rendering and synchronous rendering in React, Andrew Clark](https://twitter.com/acdlite/status/977291318324948992)
[2] [useTransition, React Docs](https://beta.reactjs.org/reference/react/useTransition)
[3] [useDeferredValue, React Docs](https://beta.reactjs.org/reference/react/useDeferredValue)
