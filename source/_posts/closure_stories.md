---
title: Unraveling the Mystery Behind Closure with Stories
date: 2023-03-02 20:30:37
tags: JavaScript Closure
---

Before I explain the concept of closure, let me share a story with you.

The word "closure" evokes a tender memory of my visit to Manchester in 2016.

During my stay, I had the pleasure of living in an Airbnb home of an 80-year-old lady. The old lady was living alone, and her best friend was a cat named Mavis. In 2016, she did not have a smartphone, and her favorite form of entertainment was listening to the radio.

![image](https://user-images.githubusercontent.com/51183663/222784849-cb99ee40-673d-4e0f-a5c9-35bc510b6f62.png)

Today, I can still envision her silhouette moving gracefully through the kitchen, preparing breakfast while enjoying her favorite radio program.

Times have certainly changed, yet this elderly lady held onto habits from decades ago until today. 

**Now, you may wonder, what does this have to do with closure?**
Well, let's take a look at the concept of closure:
> Closure is when a function is able to remember and access its lexical scope even when that function is executing outside its lexical scope.
> -- You Don't Know JS (1st edition)

**Have you noticed the similarity between an elderly lady who loves listening to the radio and a closure function remembering its original lexical scope?**

In the story I shared earlier, the elderly lady *(a closure function)*  was born in the 1930s *(its lexical scope)* and developed the habit of getting news from the radio during the time she was born.

Later, when she entered the 2010s *(another lexical scope)*, smartphones had already invaded the lives of almost everybody, but she still retained the habit that she developed in the 1930s——listening to the radio *(access its lexical scope even when that function is executing outside its lexical scope)*.

Now, let's go back to the concept of closure in JavaScript. A closure often has the following characteristics:
* A function was born in a lexical scope (i.e., the lexical scope where the function was defined).
* This function remembers and references variables from the birth scope.
* When this function runs in other scopes, it still uses certain variables from the birth scope.

To illustrate this with code, it would look like this:
```js
function year1930s() {
  const popularMedia = 'Radio';
  return function ladyS() {
    console.log(`${popularMedia} is my favorite way to get news.`);
  }
}

const ladyS = year1930s();

function year2010s() {
  const popularMedia = 'Smart phone';
  ladyS(); 
}

year2010s(); // "Radio is my favorite way to get news."
```

To delve a bit deeper, as the closure function is still using variables from the birth scope, these variables cannot be destroyed and still occupy memory (like after the function year1930s is called, the variable popularMedia is still there).

Yeah, in real life, though it’s already the 2020s, home radios haven’t completely disappeared from this world, due to some people’s love for nostalgia.

Of course, there are many other examples that can illustrate how closure works. For instance, many immigrants still have a strong affinity for their hometown food, no matter how long they have lived abroad.

To describe this with code, it could look like this:
```js
function lifeInHometown() {
  const popularFood = '🌶️ beef noodles';
  return function kidQ() {
    console.log(`${popularFood} is my favorite food.`);
  }
}

const kidQ = lifeInHometown();

function lifeInUK() {
  const popularFood = 'Fish and chips';
  kidQ();
}

lifeInUK(); // "🌶️ beef noodles is my favorite food."
```

Let’s recap the concept of closure:
- When there is a closure, it must involve two different lexical scopes:
  - one is the birth lexical scope A, where the function was defined.
  - the other is the lexical scope B, where the function is called later.
- Even if the function leaves its birthplace closure A and runs in another lexical scope, it still remembers some variables defined in A.

By now, you should have a grasp of what closure is.

However, sometimes when people talk about closure, they are not referring to the precise concept of closure. For instance, you might hear people mentioning the term 'closure,' but what they're actually referring to is:

*They have a function that does not reference data from the lexical scope they expected.*

An example of this is Stale Closure, which often appears in React hooks and is a pain for many beginners. In the next blog post, I will discuss the Stale Closure issue in React hooks in more detail.
