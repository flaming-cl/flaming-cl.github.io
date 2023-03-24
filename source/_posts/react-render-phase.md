---
title: React Source Code Little by Little 3 - Render Phase
date: 2023-01-13 08:41:00
tags: React
---
## Recap: Time Sliced Work Loop
From the [time slicing](https://flaming-cl.github.io/post/time-slicing-react) post, we have known React implements an interruptible render phase by a conditional while loop: once <abbr>shouldYield</abbr> turns True, the current reconciliation job can be paused, leaving the main thread to more prioritized jobs, like a user event.
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
Today we are going to talk about the render phase of React, which is implemented via <abbr>performUnitOfWork</abbr> inside each work loop.
<details><summary><b>Question: Do you think where we should put our code for the commit phase?</b></summary>
<p>
We should put it outside the breakable while loop.      
This is because the commit phase can not be paused: we do not want to see our App rendered with some half updated states.

```js
function workLoop(deadline) {
    let shouldYield = false
    while (workInProgress !== null && !shouldYield) {
        performUnitOfWork(
            workInProgress
        )
        shouldYield = deadline.timeRemaining() < 1
    }
    commitWork(); // the commit phase
    requestIdleCallback(workLoop)
}
```
</p>
</details>

---

## The Render Phase
<abbr>PerformUnitOfWork</abbr> (the render phase) mainly does two things: beginWork and completeWork. Its code could be written like this:
```javascript
let workInProgress = null;

function performUnitOfWork(fiber: FiberNode) {
    const next = beginWork(fiber);
    fiber.memorizedProps = fiber.pendingProps;

    if (next === null) {
        completeUnitOfWork(fiber);
    } else {
        workInProgress = next;
    }
}
```
### An Overview of PerformUnitOfWork
Before understanding key responsibilities of <abbr>performUnitOfWork</abbr>, we first have an overview about how React calls <abbr>performUnitOfWork</abbr> to reconcile a Fiber tree.

![punitofwork](https://user-images.githubusercontent.com/51183663/213318797-f18c1fa2-6c65-4ed7-aa29-bd7eea58ff9e.png)

### beginWork
### What does beginWork do?
Each time, <abbr>performUnitOfWork</abbr> calls <abbr>beginWork</abbr> with the WIP (work in progress) fiber node to do the reconciliation job and return a reconciled child Fiber node (return null if no child). 
### Execution order of beginWork calls

  The execution order of beginWork calls is similar to a preorder traversal of a tree, like succession to a monarchy's throne:
  
  > 1st kid ➡ 1st grandchildren ➡ 1st great-grandchildren ➡ ... ➡ the 1st no-kid offspring
  > ➡ second in line sibling ➡ 1st kid of second in line sibling ➡ ...

  <details><summary><b>True or False: The 1st Fiber node enters <abbr>completeWork</abbr> is the deepest node in a Fiber tree. </b></summary>
          <p>
          False. The first Fiber node enters <abbr>completeWork</abbr> is the left most leaf node in an App.
          </p>
  </details>   

### After beginWork
When <abbr>beginWork</abbr> finishes its work, we update the current props in the Fiber node.
The next step depends on whether the current Fiber node has a child node:
- child ✅, <abbr>performUnitOfWork</abbr> will pass the child to the next <abbr>performUnitOfWork</abbr> call.
- child ❌, <abbr>performUnitOfWork</abbr> will leave the current node to <abbr>completeUnitOfWork</abbr>.

### completeUnitOfWork
Then, <abbr>performUnitOfWork</abbr> calls <abbr>completeUnitOfWork</abbr>:
```js
function completeUnitOfWork(fiber: FiberNode) {
    let node: FiberNode | null = fiber;

    do {
        completeWork(node); 
        const sibling = node.sibling;

        if (sibling !== null) {
            workInProgress = sibling;
            return;
        }

        node = node.return;
        workInProgress = node;
    } while (node !== null);
}
```
Inside <abbr>completeUnitOfWork</abbr>, we first call <abbr>completeWork</abbr> to create a DOM instance for the WIP fiber node and handle its effects/flags. After that, the while loop has 3 options:
- Option 1, the WIP node has a sibling node ✅: <abbr>beginWork(sibling)</abbr>
- Option 2, sibling node ❌ , parent node ✅ : <abbr>completeWork(parent)</abbr>
- Option 3, sibling node ❌ , parent node ❌ : stop the work loop 
  
### Test your understanding
Now we have had an overview of the reconciliation work loop.
To test how you understand what we have learned, give it a whirl on the following question.
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
<summary>
<b>Question: Please list all the beginWork and completeWork calls of this App during a render phase in sequence.</b>
For example, 1. beginWork(hostFiberRoot) 2. beginWork(App) 3. beginWork(div)
</summary>


![render_path](https://user-images.githubusercontent.com/51183663/212486682-a1a58b17-0206-419a-9d7c-945eb9918afc.png)



Note: DIV is a host component. "little by little" is a text component.

### Interview Trick
You may wonder why we spend so much time on a general understanding of the render phase. By understanding it, you can solve many interview questions on the rendering path of an APP.

Let us look at an interview question:
```javascript
import React, { useEffect } from "react";

function A() {
  useEffect(() => {
    console.log("Mount A");
  }, []);
  console.log("A");
  return <B />;
}

function B() {
  useEffect(() => {
    console.log("Mount B");
  }, []);
  console.log("B");
  return <C />;
}

function C() {
  useEffect(() => {
    console.log("Mount C");
  }, []);
  console.log("C");
  return null;
}

function D() {
  useEffect(() => {
    console.log("Mount D");
  }, []);
  console.log("D");
  return null;
}

function App() {
  useEffect(() => {
    console.log("Mount App");
  }, []);
  console.log("App");
  return (
    <div>
      <A />
      <D />
    </div>
  );
}

export default App;
```
### What does the above code snippet output by calling console.log?

```
App    
A    
B    
C    
D     
Mount C    
Mount B     
Mount A    
Mount D   
Mount App  
```

- Console.log("X") is called when we enter a component. 
> Orders of entering a component are the same as how we call beginWork (like a preorder traversal).
- Console.log("Mount X") inside useEffect(() => {}, []) is called after a component is rendered. 
> Orders of rendering a component are the same as how we call completeWork (like a postorder traversal).


But can you tell the reasons for your answer? It is time to take a closer look at beginWork and completeWork.
