---
title: Single-threaded JavaScript
date: 2023-02-08 14:33:14
tags: JavaScript
---
### Single-threaded JavaScript
The JavaScript engine is single threaded.

This means it only does one thing at a time in the call stack, like the only phone booth in town that only allows one person to make phone calls.

Also, function calls that are waiting to be executed are like people who are waiting outside a booth for phone calls.

![image](https://pbs.twimg.com/media/FoeFDZfX0AA1Fqn?format=jpg&name=medium)
### Sync and Aysnc Events
In JavaScript, we have sync and async calls.

When it comes to our phone booth story, we also have two kinds of people:
- People who can start their phone call immediately (sync)
- People who just stay in the booth , and wait for others to be ready and start a call (async)

No one would like to be kept waiting outside of a phone booth, but only to see someone stay in the booth doing nothing.

An efficient JavaScript engine also does not allow async functions to occupy the call stack and have sync ones to wait.

### Callback Queue and Call Stack
So the JavaScript engine will let the sync calls to stay in the call stack, and puts async calls aside for a while.

For those async calls, once they are ready to be executed, they will be pushed into a callback queue, waiting to be back to the call stack.
Right, they are not pushed into the call stack immediately. Instead, they will be back until the call stack has finished its existing sync calls.

I think this makes sense, because we do not want to messed up async calls and sync calls.

When an async callback is ready, it is possible that there are still some sync tasks waiting to be execuated in the call stack. At this moment, if we allow a ready async task to jump the queue, you may get confused why a async call is execuated before your sync calls.
![image](https://pbs.twimg.com/media/FoeFEsiXgAE7PxT?format=png&name=900x900)
