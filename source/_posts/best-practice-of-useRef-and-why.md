---
title: Best Practice of using ref in React hooks
date: 2023-02-28 15:54:29
tags: React
---

In this article, we'll be discussing a best practice for using useRef in React, as well as delving into the reasons behind it.

Let's first look at a piece of code. What will be logged by this code in the end?
```js
import React, { useState, useRef, useEffect } from 'react';
import ReactDOM from 'react-dom'

function App() {
  const [show, setShow] = useState(true)
  return <div>
    {show && <Child unmount={() => setShow(false)} />}
  </div>;
}

function Child({ unmount }) {
  const isMounted = useIsMounted()
  useEffect(() => {
    console.log(isMounted)
    Promise.resolve(true).then(() => {
      console.log(isMounted)
    });
    unmount(); 
  }, []);

  return null;
};

function useIsMounted() {
  const isMounted = useRef(false);

  useEffect(() => {
    isMounted.current = true;
    return () => isMounted.current = false;
  }, []);

  return isMounted.current;
}

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App/>)
```
<details><summary><b>Answer (click me)</b></summary>
<p>

```js
// mount 
false
// update
false
```

```jsx
function Child({ unmount }) {
  const isMounted = useIsMounted() // mounting
  useEffect(() => {
    console.log(isMounted) // mounted
    Promise.resolve(true).then(() => {
      console.log(isMounted) // update
    });
    unmount(); // called when mounted, cause an update
  }, []);

  return null;
};

function useIsMounted() {
  const isMounted = useRef(false); // mounting

  useEffect(() => {
    isMounted.current = true; // mounted
    return () => isMounted.current = false; // cleanup function called during next update
  }, []);

  return isMounted.current; // mounting
}
```

</p>
</details>

## Why the above code does not work as expected?
### Primitive V.S. Reference Data type in JavaScript
![image](https://user-images.githubusercontent.com/51183663/221996628-e8465f1a-21ec-4f42-838d-a6b41ad17047.png)
When you create a copy of a primitive value, such as a string or a number, it is completely independent of the original value. In other words, changing the copy will not affect the original value in any way.

However, when you make a copy of reference data, such as an object or an array, it's like duplicating a key to a house. If you use this copied key to enter the house and make any changes to the interior, these modifications will be reflected for anyone else who also has access to this house.

### How primitive data affected the `useIsMounted` hook?
The value returned by ```useIsMounted``` is a snapshot of a primitive value during mounting, and any changes made to the ref after that moment will not affect this initially returned value.

It's worth noting that `refs` does not automatically trigger a re-rendering. As a result, the initial value of isMounted```false``` is called twice.

### Closure in hooks
You may also be curious about why `console.log(isMounted)` called with `Promise` is still false. To understand this, you need a little background knowledge about the execution context of each effect in React.

You can think of each render in React as a layer of dreams in the movie "Inception": 
*Data that `useEffect` can directly access is not infinite in time and space.*
It can only read the execution context that was created, when the `useEffect` was called.

So, although each update cycle in React will create a new execution context, some hooks might still refer to its old context.

This is why the Promise `console.log(isMounted)` still showed the initial `isMounted` value, even after `unmount()` triggered a new update cycle.

## How should we modify the above code to make it work as expected?
Before answering this question, let's do another quiz:    
What will be printed after runing the below code?
```js
import React, { useState, useRef, useEffect } from "react";
import ReactDOM from 'react-dom/client';

function App() {
  const [show, setShow] = useState(true);
  return <div>{show && <Child setShow={setShow} />}</div>;
}

function Child({ setShow }) {
  const isMounted = useIsMounted();

  useEffect(() => {
    console.log(isMounted.current);
    setShow(() => {
      console.log('update state');
      return false;
    });
  }, []);

  return null;
}

function useIsMounted() {
  const isMounted = useRef(2);

  useEffect(() => {
    isMounted.current += 3;
    return () => {
      isMounted.current *= 2
      console.log(isMounted.current);
    };
  }, []);

  return isMounted;
}

const root = ReactDOM.createRoot(
  document.getElementById('root')
);
root.render(< App />)
```
<details><summary><b>Answer (click me)</b></summary>
<p>

```js
// mounted
5
"update state"
// after state updated
10
```

</p>
</details>

### Explanation
Instead of returning the `isMounted.current` value, we are now returning the entire `isMounted object`. This means, although our reference key (the ref object) was created during mounting, we still can refer to an updated version of the `isMounted ref` in the componentDidMount lifecycle.

In this case, the logged `isMounted.current` values are as expected:
- **5**: After mounting, the `isMounted.current` value is 5, which is updated by the `useEffect` function in the custom hook.
- **10**: When `setShow` is triggered, it triggers a re-render of the App component, which causes the Child component to be unmounted and the cleanup function of `useIsMounted` to be called.
 
(**A tip on useEffect cleanup**: React will call your cleanup function each time before the Effect runs again, before the related component unmount)

As a result, isMounted.current = 10    

## A secret behind `useRef`
The ref returned by `useIsMounted` references the same ref object during each rendering (to understand it, try to log `isMounted` in the above code).
```typescript jsx
import React, { useState, useRef, useEffect } from "react";
import ReactDOM from 'react-dom/client';

function App() {
  const [show, setShow] = useState(true);
  return <div>{show && <Child setShow={setShow} />}</div>;
}

function Child({ setShow }) {
  const isMounted = useIsMounted();

  useEffect(() => {
    console.log(isMounted);
    setShow(() => {
      console.log('update state');
      return false;
    });
  }, []);

  return null;
}

function useIsMounted() {
  const isMounted = useRef(2);

  useEffect(() => {
    isMounted.current += 3;
    return () => {
      isMounted.current *= 2
      console.log(isMounted);
    };
  }, []);

  return isMounted;
}

const root = ReactDOM.createRoot(
  document.getElementById('root')
);
root.render(< App />)
```

Can you take a guess on what will be logged by this code?
<details><summary><b>Answer (click me)</b></summary>
<p>

```js
{current: 10}
'update state' 
{current: 10}
```

</p>
</details>

### Explanation
To understand this, let's look at the source code of updating refs in React:
```typescript
function updateRef<T>(initialValue: T): {current: T} {
  const hook = updateWorkInProgressHook();
  return hook.memoizedState;
}
```
As you can see, the updated ref object always refers to the initial one.
This explains why you see `{current: 10}` before 'update state':
*The ref always points to the same object, and `{content:10}` is showing its most recent value.*
If you want a ref to reflect data changes more accurately, you can convert the ref object to a string or call ref.current.

## Recap: best practice of returning a ref in custom hooks
1. return the ref object itself
2. call ref.current to access the latest value

