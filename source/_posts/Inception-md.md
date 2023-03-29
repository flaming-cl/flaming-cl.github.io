---
title: Grasping UseEffect Stale Closure with Inception
date: 2023-03-18 23:45:30
tags: React
---

The first time I sought a deeper understanding of useEffect was due to——an unexpected bug in a React project at work. Later, I found this was because I did not grasp the essence of this hook.

At its core, useEffect is a hook that allows you to perform data operations after each rendering of a component. If you don't want it to be called every time, you can pass an array of dependencies to it:
- Passing an array with a specific state. This means that **you want your operation to be performed only when a certain state is updated**.
```js
useEffect(() => {}, [someState]);
```
- Passing an empty array. This means that **you want certain operations to run after the initial render of the component**.
```js
useEffect(() => {}, []);
```

But what I was not aware of was **`useEffect(() => {}, [])` can only access the data produced by the initial render**. It cannot access the updated state from any subsequent renders. In this case, one may encounter what's known as **a stale closure in React useEffect hook**.

### Stale Closure
Stale closure—don't be intimidated by this buzzword. Let's start with something light and fun: movie Inception.

*"Oops, just a heads-up! I'll be sharing spoilers about the movie's plot. If you'd prefer not to know, feel free to watch the movie first and then return to enjoy this blog post."*

Inception is one of my favorite movie. Its protagonist can create and navigate multiple layers of dreams using dream manipulation skills, ultimately implanting a belief-like idea into someone's mind.

You can think of React as a dream scene architect, where each time you call `setState`, it's like pressing a button to construct a new dream layer.

So if you trigger `setState` after the App's first render, you're signaling React to initiate a new layer of rendering snapshot. 
However, **even though a new rendering has been completed, there can still be functions referring to the data from the first rendering**. Here comes in what we called the `useEffect` stale closure pitfall, as seen in this code snippet:
```js
export default function App() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const timer = setInterval(() => {
      console.log(count);
    }, 1000);
    return () => clearInterval(timer);
  }, []);

  const increment = () => {
    setCount(count + 1);
  };
  
  return (
    <div>
      <button id="btn" onClick={increment}>
        click me
      </button>
    </div>
  );
}
```

What will be logged after clicking the button 3 times?
`always 0`

In this code snippet, the callback function in `useEffect` is trapped within the first rendering state snapshot, unable to access data from subsequent renders.

### Explanation with Movie Inception
**How should we understand this?**

Let's go back to the movie for an explanation.
As the story progresses, the movie Inception features 3 scenes of dream simultaneously: 
- In the newest dream layer (the 3rd layer), Cobb has arrived at the latest snowy mountain scene.
- 2nd layer: Cobb's companion Arthur stays in the second layer to complete his mission.
- 1st layer: their companion Yusurf remains in the first layer racing against time.

![New Project (1)](https://user-images.githubusercontent.com/51183663/226151016-69e59a76-0f19-41a6-a5b3-45f42e4f8262.png)

**Here are the similarities between Inception's dreamscapes and React's renders** (disclaimer: there are actually many differences, but for now, let's focus on the similarities that help you understand stale closures) are as follows:
- **React: States in different React renders are independent of one another.**
- Inception: Each dream has its own independent scene. We can think of each dream layer as a snapshot of the React state. Due to the independence between different dreamscapes, some characters with specific mission requirements can only stay within a specific scene.
- **React: After the first render in React, a timer triggered by useEffect(() => {}, []) can only use the states related to the first render, and cannot access the latest state.**
- Inception: Yusurf in the first dream layer remains in that layer, and never access the subsequent dream layers, like the hotel or the snow mountain.

By drawing this comparison, hopefully you can better understand the concept of stale closures in React hooks: **Stale closure refers to a situation where a callback function inside a useEffect hook cannot capture updated state values.** That callback is like a character trapped in an old dreamscape.


### Reasons behind independent rendering states
Inception creates 3 completely different scenes to make the dream experience more immersive and to make the dreamer firmly believe in the implanted idea. 

**But why does React make the state of each render independent?**

This is due to React's functional programming nature. 
By doing this, React ensures that, at any given moment, a component's state is immutable. This makes it easier for developers to predict the behavior of a React App.

For example, by using `useEffect(() => {}, [])` to perform a certain operation, you're kind of telling React: "Hey, this hook's callback is just about the first render". So React, being a good friend and following the data immutability rule, happily hands over data only from that initial render to you.


### Fix stale closure in useEffect
How to fix the issue in the timer code snippet, if we want the timer to log the latest count value?

**To solve a problem, let's first understand what the issue is:**
Stale closure occurs when a useEffect callback can never access the updated states.

Therefore, our solution should be to focus on how to provide updated data to this callback.

#### Solutions
##### Idea 1: allowing the callback to read each updated state snapshots
- **Action**: include the count state as a dependency of this useEffect hook
- **Cons**: you are creating and destroying a timer for each new count.
```js
function App() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const timer = setInterval(() => {
      console.log(count);
    }, 1000);
    return () => clearInterval(timer);
  }, [count]);

  const increment = () => {
    setCount(count + 1);
  };
  return (
    <div>
      <button id="btn" onClick={increment}>
        click me
      </button>
    </div>
  );
}
```
##### Idea 2: keeping the callback in the first state snapshot, but providing it with a ref .
- **Action**: use a ref to store the count value (Unlike states and effects, refs are objects that are passed into all rendering snapshots, they are mutable)
- **Pros**: you are not creating and destroying a timer for each new count
- **Cons**: updates on countRef.current can not be displayed in time to your App (any updates on ref.current will not trigger re-rendering in React).
```js
function App() {
  const countRef = useRef(0);

  useEffect(() => {
    const timer = setInterval(() => {
      console.log(countRef.current);
    }, 1000);
    return () => clearInterval(timer);
  }, []);

  const increment = () => {
    countRef.current += 1;
  };
  return (
    <div>
      <button id="btn" onClick={increment}>
        click me
      </button>
    </div>
  );
}
```

