---
title: Data Abstraction from SICP
date: 2023-03-27 20:14:53
tags: Functional-Programming
---

### Data Abstraction
In functional programming, data abstraction is a technique that uses functions to hide the details of data structures, and provides a well-defined interface for accessing the data. In this way, the rest of the program does not have to deal with the details of the data structure, but only with the abstraction layer provided by the functions.

By encapsulating the data structure, it also becomes easier to modify it in the future. Changes to the data structure can be made by adjusting the implementation of the functions that encapsulate it, without affecting the rest of the program.

This article will show you how to use data abstraction to minimize the impact of unpredictable data changes to your program.

### Example: Data abstraction V.S. a plain data handling method
> Write a function `midpoint_segment` to find out the middle point of a certain segment.

First, let's take a look at how to implement `midpoint_segment` with data abstraction.
```js
function make_point(x, y) {
  return {
    x: x,
    y: y
  };
}

function x_point(point) {
  return point.x;
}

function y_point(point) {
  return point.y;
}

function make_segment(start, end) {
  return {
    start: start,
    end: end
  };
}

function start_segment(segment) {
  return segment.start;
}

function end_segment(segment) {
  return segment.end;
}

function midpoint_segment(segment) {
  const start = start_segment(segment);
  const end = end_segment(segment);
  const mid_x = (x_point(start) + x_point(end)) / 2;
  const mid_y = (y_point(start) + y_point(end)) / 2;
  return make_point(mid_x, mid_y);
}

function print_point(point) {
  console.log(`(${x_point(point)}, ${y_point(point)})`);
}

const segment = make_segment(make_point(1, 2), make_point(5, 6));
const midpoint = midpoint_segment(segment);
print_point(midpoint);
```
In this case, we simply wrap the data definitions and retrieval operations with functions. By doing so, we avoid exposing the point data to the implementation logic of midpoint_segment. You may think this is redundant and cumbersome, but let's not jump to conclusions yet and keep reading.

Now we use a plain method to implement `midpoint_segment`.
```js
function Point(x, y) {
  this.x = x;
  this.y = y;
}

function Segment(start, end) {
  this.start = start;
  this.end = end;
}

function midpoint(segment) {
  const start = segment.start;
  const end = segment.end;
  const mid_x = (start.x + end.x) / 2;
  const mid_y = (start.y + end.y) / 2;
  return new Point(mid_x, mid_y);
}

function print_point(point) {
  console.log(`(${point.x}, ${point.y})`);
}

const start = new Point(1, 2);
const end = new Point(5, 6);
const segment = new Segment(start, end);
const mid = midpoint(segment);
print_point(mid);
```

It looks like our plain method code is concise and readable.

**However, is the code of the plain method easier to maintain than the code using the data abstraction method?**
#### Potential problems with plain implementation
Suppose in the future, the data structure of a point becomes an array `[2, 4]`. 

Now, we need to make changes to the implementation of the plain method in all the code that uses `point.x` and `point.y`.

However, experienced programmers know that modifying code in multiple places in a complex project is a **dangerous** signal. No one knows what unknown effects will happen when we let the butterfly flap its wings in a certain piece of code.

A better way is to try our best to avoid make many changes to our existing code. 

#### Why data abstraction helps you to build robust programs
Now, we look at our code implemented with data abstraction.

You will find you don't have to search through the whole project to find out how many times you used `point.x` or `point.y`. 
Although you still make a small number of changes, you at least have an overall understanding of its impact to the whole project, reducing the possibility of unknown bugs caused by code modifications.

```js
function make_point(x, y) {
  return [x, y];
}

function x_point(point) {
  return point[0];
}

function y_point(point) {
  return point[1];
}

function make_segment(start, end) {
  return {
    start: start,
    end: end
  };
}

function start_segment(segment) {
  return segment.start;
}

function end_segment(segment) {
  return segment.end;
}

function midpoint_segment(segment) {
  const start = start_segment(segment);
  const end = end_segment(segment);
  const mid_x = (x_point(start) + x_point(end)) / 2;
  const mid_y = (y_point(start) + y_point(end)) / 2;
  return make_point(mid_x, mid_y);
}

function print_point(point) {
  console.log(`(${x_point(point)}, ${y_point(point)})`);
}

const segment = make_segment(make_point(1, 2), make_point(5, 6));
const midpoint = midpoint_segment(segment);
print_point(midpoint);
```
Now, do you prefer to individually wrap each cookie or leave them scattered together?

![cookies](https://user-images.githubusercontent.com/51183663/229636030-e3139501-6c8c-46a8-a568-a5961740a6a1.png)

My Tip: I still don't think one should stubbornly adhere to a certain method.
But next time if I anticipate significant changes to a data structure in the future, I will consider using data abstraction to encapsulate the data, for more resilient code.
