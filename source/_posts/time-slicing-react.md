---
title:  React little by little 1 -  Time Slicing
date: 2022-09-28 18:49:47
tags: React
---

You may have heard people saying "React is fast" as it uses virtual DOM.
But is virtual DOM the only thing you can think of when talking about React performance?
In fact React has done many things to handle performance issues.

Today I will talk about Time Slicing.
A technique React uses to solve performance bottleneck.

## Event Loop and CPU bottlenecks
### Event Loop
Modern browsers run a continuous event loop at a certain frequency - e.g. 60Hz (refresh 60 times a second). For smooth and responsive experience, browsers try to complete tasks within each event loop execution, aka a frame.

![event-loop-frame](https://user-images.githubusercontent.com/51183663/209892101-70af523b-4bf5-45ba-9aef-b01ac11f9b70.png)

For 60Hz refresh rate devices, each execution frame has a budget of 16.66ms (1 second / 60).

### CPU bottlenecks
Within such a shoestring budget, browsers have to handle a pipeline of work like this:

![frame_pipeline](https://user-images.githubusercontent.com/51183663/209890234-985a0618-b856-414d-999e-f41823756b9b.png)

If any step in this pipeline needs more than 16.66ms, no time will be left for the upcoming work in this frame.
This means we will not have time for style calculations, layout, paint and compositing, when we spend too much time on JavaScript jobs.
When this lasts for 2 or 3 seconds, users will feel your website is slow.

### How does React avoid this scenario?
React sets up an interval for running JavaScript tasks in each frame. 
When running out of this interval, React will pause its current tasks and give back control to the main thread, letting it perform high priority tasks (painting or user events).
After the main thread has finished prioritized jobs, React goes back to where it stopped and continues working.

As you may have noticed, the key here is to make React interruptible.

## Time Slicing and Interruptible React 
Assume we have a large application with 500+ components to render.
### While rendering these components, how does React ensure responsiveness for keyboard or mouse events?

Before React 16, the render phase was synchronize and uninterruptible.
The browser therefore would be easily occupied by CPU-hungry tasks,
and unable to give prompt responses to user events.
Take a look at [this video](https://www.youtube.com/watch?v=nLF0n9SACd4) to see what CPU-heavy tasks could be like.

### So, how did React overcome this?

Answer: Time Slicing.

Here is a [picture](https://twitter.com/acdlite/status/977291318324948992) of time slicing in React (idea of Andrew Clark from the React core team).

![time_slicing](https://user-images.githubusercontent.com/51183663/211950568-f4b85e7a-91d6-4a5a-9b97-c949a14e4282.png)

### Interruptible render phase
After version 16, React started using Fiber reconciler to ensure the render phase is interruptible. It breaks down the render phase into time slices so that it can pause current work to run more urgent tasks, like user inputs.
As shown in the above picture, time slicing enables browser events to cut in line during the render phase. This helps React to avoid "hiccup" moments.

### Uninterruptible commit phase
Unlike the render phase, the commit phase can not be interrupted.
You can think of these two like a movie screenplay and a movie on show.
When a movie is on (the commit phase), theatres will not suddenly stop playing or change its order of scenes. Just like React will not draw some semi-calculated UI or suddenly stop the commit phase.

## Interruptible Work Loop
Before version 16, React reconciler is implemented by call stack recursion. However, stack Reconciliation will not allow pauses on a running task during the render phase.
As mentioned earlier, a high performance render phase is time sliced and interruptible. To implement this, we can change the call stack recursion to conditional while loop recursion:
```js
function workLoop(deadline) {
    let shouldYield = false
    while (workInProgress !== null && !shouldYield) {
        // next workInprogress is set within performUnitOfWork
        performUnitOfWork(
            workInProgress 
        )
        shouldYield = deadline.timeRemaining() < 1
    }
    requestIdleCallback(workLoop)
}
```
- while (workInProgress !== null && !shouldYield)
> This enables us to stop the workLoop, especially when there is no time left in each frame (shouldYield).  
- requestIdleCallback
> Check whether the main thread is idle for running sliced tasks.

## Fiber Reconciler
This interruptable reconciler is called Fiber reconciler in React.
To know more about React Fiber, take a look at this post first: [React Fiber Tree](https://flaming-cl.github.io/bits-refinery/2022/11/22/react-fiber-tree/).

## References
[1] [Sneak Peek: Beyond React 16, Dan Abramov](https://reactjs.org/blog/2018/03/01/sneak-peek-beyond-react-16.html)
[1] [Build your own React, Rodrigo Pombo](https://pomb.us/build-your-own-react/)
[2] [Event loop: microtasks and macrotasks, javascript.info](https://javascript.info/event-loop)
[3] [Visualization of async rendering and synchronous rendering in React, Andrew Clark](https://twitter.com/acdlite/status/977291318324948992)
[4] [Reconciliation, React official documents](https://reactjs.org/docs/reconciliation.html)
[5] [Rendering Performance, Paul Lewis](https://web.dev/rendering-performance/)
[6] [Frame timing, W3C](https://www.w3.org/TR/frame-timing/)
