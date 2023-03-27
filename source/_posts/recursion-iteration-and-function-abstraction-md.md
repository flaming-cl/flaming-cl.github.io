---
title: Recursion, Iteration and Abstraction from SICP
date: 2023-03-26 19:45:35
tags: Functional-Programming
---

I've recently been reading SICP, a book about programming philosophy. I just finished the 1st chapter yesterday and wanted to share some of my thoughts and insights.

The main points in the 1st chapter that got me thinking were:

1. Difference between Recursion and iteration
2. How to abstract a function to make it more extensible and reusable

## Recursive or Iterative Processes
To make it easier to understand, I'll start by discussing a topic that many frontend developers are familiar with – React.

### Recursive Process
The diff algorithm in React prior to version 16 is a prime example of a recursive process.
 
**Diff Algorithm**: To identify the changes in the Virtual DOM, React starts at the root node and recursively traverses down the tree. This process consists of expansion and contraction phases.

#### Expansion (beginWork)
In the expansion phase, we traverse the virtual DOM tree from the root node, exploring child nodes step by step.

#### Contraction (completeWork)
During contraction, we return to the parent node after traversing all child nodes, then proceed to a sibling or parent's parent.

#### A Story to Help You Visualize the Process
If the above explanation is still a bit unfamiliar to you, let me tell a story to help you visualize the process:

![image_treasure_hunt](https://user-images.githubusercontent.com/51183663/227833411-f0bd9fd7-5451-4deb-b5fe-1c08484c4091.png)
- You're playing a game where an unknown number of treasures are hidden within an underground maze's various rooms. 
- Without a map, you venture deeper into each room to explore. 
- At dead-ends, you backtrack to the nearest fork until you eventually return to the maze's entrance to end the game.

![image_treasure_hunt_rooms](https://user-images.githubusercontent.com/51183663/227833542-14ab3954-389c-4c8b-a3a4-8e7ea0ab303c.png)

In this context, delving into deeper rooms symbolizes an expanding process, akin to exploring new territories. Conversely, backtracking from dead-ends represents a contracting process.
#### Unaware of Overall Gain
Now let us discuss another feature of recursion: **before completing the entire recursive process, we are unaware of the overall results, as we only focus on local problems.** 

Going back to the treasure hunting story. 
While exploring the maze, you are given a magical pocket to conceal all the treasures collected. Only upon completing the adventure, the pocket reveals your accumulated riches. 

In this case, the uncertainty of your treasure hunting gains mirrors the ambiguity of progress in recursion, where a clear result emerges only upon task completion.
#### Uninterruptible
**Another problem with recursion is that it is hard to be interrupted and resumed.**

Recursion can be difficult to interrupt and resume, making it hard to pause a task or prioritize more urgent tasks. To make an expandable and contractible process interruptible, we need to combine recursive and iterative processes.

### Iterative Process
Iterative processes are repeatable and build upon previous results, allowing for interruptions, resumptions, and data recording for continuity.

Since React version 16, iteration has been using to traverse the DOM tree through a conditional loop. This enables prioritized tasks to interrupt and later resume the diff process using stored data. Also, it maintains expandable and contractible approaches to traverse a Fiber tree for detecting changes.
## Function Abstraction
I think the most interesting part of this chapter is function abstraction.

To explore the beauty of it, let's start with a simple example, and gradually improve it upon small initiatives. 
> Create a composite function that takes a number as input, and returns the cube of the number after adding 1.
```js
const incAndCube = (x) => (x + 1) * (x + 1) * (x + 1);
```
Here, repeatedly having `x + 1` does not sound good. So let's abstract it into a function. 
```js
const inc = (x) => x + 1;

const incAndCube = (x) => inc(x) * inc(x) * inc(x);
```
### Higher-order Function
Right now, though our code looks a bit better, we are still repeating `inc` 3 times. To improve this, let's pass `inc` as a parameter of `cube`, to make `cube` a higher-order function and avoid repetitiveness. By using a higher-order function in this case, we can create a more flexible and maintainable solution.
```js
const inc = (x) => x + 1;

const cube = (x) => x * x * x;

const incAndCube = (x) => cube(inc(x));
```
In this way, if future requirements involve adding 1 then squaring, we can easily switch `cube` to `square` without changing it in three places.

You might feel like it doesn't make any difference. But don't worry, the process of iterative improvement often starts with small steps that may seem trivial. By continuously refining our code, we can achieve more extensible and reusable solutions.

### Recursion
You might feel that the `cube` function does not appear very extensible: what if the requirement changes to multiplying a number by itself 999 times?

To make the cube function more extensible, we can create a recursive `repeat` function which accepts a parameter representing the number of repetitions.

#### Expansion
**Let's think about this question: what will happen if we use `repeat` to multiply a number by itself 999 times?** 
We will first come to the expansion phase, as it will generate a massive expression involving 999 multiplication operations gradually. For example, `x * x` for once, `(x * x) * x` for twice.

#### Contraction
After finished the expansion phase, we start to shrink the large expression to a single number. We can do this by having `(result of each multiplication) * (next number)`.

With a clear understanding of expansion and contraction phases, now we can write a recursive `repeat` function to achieve greater flexibility and extensibility.
```js
const inc = (x) => x + 1;

const repeat = (x, n)  => {
  if (n === 0) {
    return 1;
  }
  return x * repeat(x, n - 1);
}

const cube = (x) => repeat(x, 3);

const incAndCube = (x) => cube(inc(x));
```
#### Explanation with Code
- **Expansion phase**: The `repeat` function, when called with `x` and `n`, will recursively call itself until `n` is reduced to `0`. During this process, it creates an expression that multiplies `x` by itself `n` times, e.g., `x * x` for once, `(x * x) * x` for twice, and so on.
- **Contraction phase**: When `n` reaches `0`, the base case is met, and the recursion starts unwinding. As it returns from each recursive call, it contracts the expression by successively multiplying the result of each multiplication by the next number. Eventually, it arrives at the final result, which is `x` multiplied by itself `n` times.

#### Explanation with Inception
If you want a more intuitive understanding of this process, think of the movie Inception.
- **Expansion**: main characters delves deeper into dreams, layer by layer.
- **Base case met**: at the moment the mission is completed, the base case is reached.
- **Contraction**: everyone employs a 'kick' to sequentially fall through dream layers, ultimately returning to reality (layer 0).

Now we have successfully abstracted the `cube` function and made it more extensible. 🎉🎉

### Currying
Wait, are we finished? It seems like we haven't seen any cool techniques yet. For example, where is currying?

We definitely can incorporate currying to our existing code. First, let's try a simple currying by making `incAndCube` returns a function.
```js
const incAndCube = () => x => cube(inc(x));
```

Now, we incorporated currying in `incAndCube`. But sorry that this currying doesn't do anything practical in the first step, nor does it store any temporary variables. It is just for the sake of experiencing the cool currying technique.

🤔 How about we try to split `incAndCube` into 2 steps? 
- The 1st step is to obtain the function expression for the entire calculation of `x`.
- The 2nd step is to substitute x with a value. 

### Function Composition
For the 1st step, we need to write a function to compose the `inc` and `cube` functions. 
Its input is two functions, and its output is the combination of these two functions.
```js
const compose = (g, h) => x => g(h(x));
```

The above code means that we first calculate `h`, then calculate `g`.
So we can write the `incAndCube` function like below, as we need to first calculate `inc` and then calculate `cube`.
```js
const incAndCube = ()  => {
  const composedFunc = compose(cube, inc);
  return x => composedFunc(x);
}
```

🌇 Now, we finished it! Here is the complete code:
```js
const inc = (x) => x + 1;

const repeat = (x, n)  => {
  if (n === 0) {
    return 1;
  }
  return x * repeat(x, n - 1);
}

const cube = (x) => repeat(x, 3);

const compose = (g, h) => x => g(h(x));

const incAndCube = ()  => {
  const composedFunc = compose(cube, inc);
  return x => composedFunc(x);
}

console.log(incAndCube()(2));
```

### Sing a Different Tune
Functional programming offers advantages in some cases, but it's not always the best solution for every situation.

The original, straightforward definition of incAndCube with three `x + 1` multiplications is already simple and easy to understand. In cases where reusability and extensibility aren't required, this approach saves programming time and remains clear to other developers.

However, in large-scale projects with possible shifting requirements, single-purpose and straightforward functions prove more robust. Consider our basic `inc` function: if its logic changed from `x + 1` to `x + 2`, we'd have to modify the original code in three places. If the function were more much complex than `inc`, multiple updates may introduce unforeseen problems.

In conclusion, each programming method has its strengths. While functional programming might seem more advanced, it's not a one-size-fits-all solution. It's essential to choose the appropriate approach based on the specific situation and project requirements.
