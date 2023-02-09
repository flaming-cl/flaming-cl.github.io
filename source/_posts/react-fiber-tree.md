---
title: React little by little 2 - Fiber Tree
date: 2022-11-22 23:40:35
tags: React
---
> Plato said life is like being chained up in a cave, forced to watch shadows flitting across a stone wall.

We are not talking about philosophy today, but use [Allegory of the Cave](https://ed.ted.com/lessons/plato-s-allegory-of-the-cave-alex-gendler) to describe the relationship between React applications and what is under the hood:

React applications we see today are mainly results of the render phase and the commit phase.
Just like shadows in the eyes of prisoners are results of sun reflection and real world activities.

![allegory_cave_dark](https://user-images.githubusercontent.com/51183663/209894263-18de7386-087f-4a0d-8418-dba2744d016d.png)

### What is under the hood
Before we see any changes in an application, React does three things: Scheduling, Rendering and commit. What are these? In plain English, they are just:
- Scheduling 🗓️: schedule and manage tasks 
- Render 🔎 🌲 🏁: figure out changes made in React applications
- Commit ☑️ : apply these changes to browser DOM

To understand React, we will start from the render phase. To understand the render phase, we need to understand the React Fiber data structure.

In this post, I will cover 2 topics about React Fiber:
- How does React create the first Fiber node?
  - What are the main properties of a Fiber node?
- How does React build a Fiber Tree from the first Fiber node during mounting?

## Inside React Reconciliation: Fiber Trees
Frequent DOM manipulations are expensive for complicated web applications.
In this case, we had better only applies changes to the real DOM during state/props updates. 
But changes might spread all over the whole tree, to locate them, we can compare two versions (previous & current) of a virtual DOM tree. This process is called Reconciliation in React.

### Question: What is a virtual DOM tree called in React 18?
Fiber tree, a singly linked list tree.
To understand Fiber tree, we can start from the most basic unit of it, a Fiber node.

## How does React generate the first Fiber node
A Fiber node is just a plain JavaScript object.
But it may be the object you have never seen before, since it is not exposed to React users.
Let us start from something we are familiar with—React Components.

### React Components, Elements, Fiber Node
The process of building a Fiber node is the process of converting React Component JSX => a React Element => a Fiber node. 
<details><summary><b> JSX </b></summary>
<p>

> A syntax extension to JavaScript that allows writing HTML in React components

</p>
</details>

<details><summary><b> React Element </b></summary>
<p>

> The smallest building blocks of React apps, describing what to render to a React UI environment

</p>
</details>

<details><summary><b> Fiber node </b></summary>
<p>

> A basic unit of work to locate or commit changes in React applications

</p>
</details>

### From React Component to React Element
We know that React components return JSX. But what does JSX produces?
```javascript
const App = () => {
    return (
        <div>
            react
            <span>
                little by little
            </span>
        </div>
    )
}
ReactDOM.createRoot(rootElement).render(<App />);
```
JSX produces React elements.
For example, App is a React component; <App / > is a React element in the form of JSX.
If you put <App / > into [Babel](https://babeljs.io/repl), you will get:
```javascript
/*#__PURE__*/React.createElement(App, null);
```
It is just a JavaScript object:
```javascript
console.log(React.createElement(App, null));

/*
{
type: ƒ App(), // e.g. functional component, or HTML tag type like div, p, h1
key: null, // e.g. keys you set up in a React component
ref: null,
props: {}, // e.g. React component props
_store: {},
_owner: null,
}
*/
```
Take a closer look at this React Element object, and answer this question:

<details><summary><b>Can a React Element be the unit of work to do the Reconciliation job? </b></summary>
<p>

No. For Reconciliation, a React element can be too static and isolated:
- Data stored in it is comparatively static. It mainly explains what this React element looks like.
> For reconciliation, we need more dynamic data about its state/effect changes or priority scheduling.
- It does not show connections with its children/sibling/parent Elements.
> When we have finished processing it, we have no way to find the next unit of work.

For reconciliation, we need an upgraded version of React Element. That is where Fiber node comes in.

</p>
</details>

### From React Element to Fiber node
We can create a Fiber root node for the React element <App / >, by calling:
```javascript
const rootElement = document.getElementById("root");
ReactDOM.createRoot(rootElement).render(<App />);
```

To see what is inside a Fiber node (React 18.2.0), you can try this:

```javascript
const reactContainerKey = Object.keys(rootElement)[0]; // __reactContainer$cjvrzgbs4i4 
console.log(rootElement[reactContainerKey]) // Fiber root node
```

![fiber_node_log](https://user-images.githubusercontent.com/51183663/210151836-ae932fd9-cfbe-424d-85db-f3c9882cd11c.png)
To make it easier to understand, I break down properties of a Fiber node object into 4 parts:
- Basic Fiber node info
- Tree pointers
- Effect & reconciliation
- Scheduling

Here is a cheatsheet for a React Fiber node:

![fiber_node](https://user-images.githubusercontent.com/51183663/212479942-a25a466e-a095-4409-9f9e-70c20139f918.png)

At this moment, just take a glimpse at this Fiber node.
Knowing it also tells something about Reconciliation, Scheduling and its neighbor nodes is enough at this moment. 
## How does React build a Fiber tree from the Fiber root node?
Now we have created a Fiber root node.
Before mount (first time rendering), React does three things:
1) Create a Fiber root node for the whole App.
2) Create a Host Fiber node for the rootElement DOM element.
3) Link the Fiber root node with the host Fiber node, using the "current" pointer.

![beforeMount](https://user-images.githubusercontent.com/51183663/211693577-4f9e384d-ff4e-414a-8d04-8792cc986bdc.png)

During mounting, React will also create an alternative host Fiber node. 
This is different from our existing one, as its child node is not null but a tree of Fiber nodes, which correspond to each React element listed in the app.

![mounting](https://user-images.githubusercontent.com/51183663/211698358-9ab6c474-e979-4333-a59e-892d7453577d.png)
The alternative host Fiber node, along with its descendant Fiber nodes, is called the workInProgress (WIP) Fiber tree.

<details><summary><b>Look at the above pic, you may wonder: Why two host root nodes? What is double buffering? </b></summary>
<p>

Here is an illustration from [game programming patterns](http://gameprogrammingpatterns.com/):
> Imagine we are watching a play. 
> When scene 1 is finished, we will have a 5-minute scene transition, leaving the audience in dark and silence. This is normal in a real world theatre play. 
> But if we leave game players in a black screen for a few seconds, this may drive people nuts.

In both game programming and React, we hope to avoid this blank transition by a technique called [Double Buffering](http://gameprogrammingpatterns.com/double-buffer.html). 
Back to our theatre play example, we avoid the silent transition by running two stages (double buffering): 
- stage A is on show for the current scene 
- stage B is adjusting lights or removing/changing play props for the next scene

Once the current scene is finished on stage A, we immediately light on stage B to go to the next scene seamlessly. Now stage A is in dark preparing the next scene settings. 

So in React, double buffering means switching between the current fiber tree and a workInProgress fiber tree for seamless transitions between rendering.

---
</p>
</details>

We therefore have finished a simple tour about first time rendering in React.

## References
[1] [Double Buffering](http://gameprogrammingpatterns.com/double-buffer.html)
[2] [Allegory of the Cave](https://ed.ted.com/lessons/plato-s-allegory-of-the-cave-alex-gendler)
