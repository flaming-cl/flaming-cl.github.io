---
title: Data Abstraction or Adapters?
date: 2023-03-27 20:14:53
tags: Functional-Programming
---

世界是千变万化的，需求也可能会变化。我们的代码也应该能够适应这种变化。在这篇文章中，我将会讨论如何在不大量修改代码的情况下，让代码能够适应需求的变化。
我觉得 Data abstraction 和 Adapters 虽然都是应对需求变化的方法，但我们使用它们的时机是很不同的。
深思熟虑：我们在面对一个新的需求的时候，应该先考虑使用 Data abstraction，尽可能地把数据操作和业务逻辑分离开来。
亡羊补牢：而当我们的已有设计不能满足新的需求的时候，我们应该考虑使用 Adapters。

### Data Abstraction
 In functional programming, data abstraction means using functions to encapsulate data structures, and accessing the data through a well-defined interface provided by those functions. This way, the rest of the program does not have to deal with the details of the data structure, but only with the abstraction layer provided by the functions.

This approach makes it easier to modify the data structure in the future, because changes to the data structure can be made by changing the implementation of the functions that encapsulate it, without affecting the rest of the program. This helps to minimize the impact of changes to the data structure on the overall codebase, and can make it easier to maintain and extend the code in the long term.

```js
// 构造函数 make_point，以 x 和 y 为参数，返回一个点对象
function make_point(x, y) {
  return {
    x: x,
    y: y
  };
}

// 选择器 x_point，返回点对象的 x 坐标
function x_point(point) {
  return point.x;
}

// 选择器 y_point，返回点对象的 y 坐标
function y_point(point) {
  return point.y;
}

// 构造函数 make_segment，以 start 和 end 为参数，返回一个线段对象
function make_segment(start, end) {
  return {
    start: start,
    end: end
  };
}

// 选择器 start_segment，返回线段对象的起点
function start_segment(segment) {
  return segment.start;
}

// 选择器 end_segment，返回线段对象的终点
function end_segment(segment) {
  return segment.end;
}

// 函数 midpoint_segment，以线段对象为参数，返回其中点的坐标
function midpoint_segment(segment) {
  const start = start_segment(segment);
  const end = end_segment(segment);
  const mid_x = (x_point(start) + x_point(end)) / 2;
  const mid_y = (y_point(start) + y_point(end)) / 2;
  return make_point(mid_x, mid_y);
}

// 打印点的方法
function print_point(point) {
  console.log(`(${x_point(point)}, ${y_point(point)})`);
}

// 测试 midpoint_segment 函数
const segment = make_segment(make_point(1, 2), make_point(5, 6));
const midpoint = midpoint_segment(segment);
print_point(midpoint); // 输出 (3, 4)

```


```js
// 构造函数 Point，以 x 和 y 为参数，返回一个表示点的对象
function Point(x, y) {
  this.x = x;
  this.y = y;
}

// 构造函数 Segment，以 start 和 end 为参数，返回一个表示线段的对象
function Segment(start, end) {
  this.start = start;
  this.end = end;
}

// 函数 midpoint，以一个线段对象为参数，返回其中点的坐标
function midpoint(segment) {
  const start = segment.start;
  const end = segment.end;
  const mid_x = (start.x + end.x) / 2;
  const mid_y = (start.y + end.y) / 2;
  return new Point(mid_x, mid_y);
}

// 打印点的方法
function print_point(point) {
  console.log(`(${point.x}, ${point.y})`);
}

// 测试 Segment 和 midpoint 函数
const start = new Point(1, 2);
const end = new Point(5, 6);
const segment = new Segment(start, end);
const mid = midpoint(segment);
print_point(mid); // 输出 (3, 4)

```

|                    | Adapters                          | Data Abstraction                     |
|--------------------|-----------------------------------|-------------------------------------|
| **Pros**           |                                   |                                     |
| 1.                 | Allows easier integration of      | Encapsulates implementation         |
|                    | existing code with new interfaces | details, allowing for cleaner       |
|                    | or data structures.               | and more maintainable code.         |
| 2.                 | Enables reuse of existing code    | Simplifies code changes by          |
|                    | with minimal modifications.       | limiting the impact of              |
|                    |                                   | modifications to the constructor    |
|                    |                                   | and selector functions.             |
| 3.                 | Can provide a temporary           | Promotes better separation of       |
|                    | solution for integrating          | concerns and modularity in the      |
|                    | incompatible systems while        | application.                        |
|                    | working on a more permanent       |                                     |
|                    | solution.                         |                                     |
| **Cons**           |                                   |                                     |
| 1.                 | Adds an extra layer of            | Requires initial investment in      |
|                    | complexity, making code harder    | creating constructors and           |
|                    | to understand and maintain.       | selectors.                          |
| 2.                 | May not provide a long-term       |                                     |
|                    | solution, requiring more          |                                     |
|                    | refactoring in the future.        |                                     |
| 3.                 | Potentially loses some benefits   |                                     |
|                    | of data abstraction by not        |                                     |
|                    | fully encapsulating the           |                                     |
|                    | implementation details.           |                                     |

### Real world examples
### Data Abstraction
Express.js uses data abstraction in several ways, most notably in the way it represents and manipulates HTTP requests and responses. In Express.js, request and response objects are abstracted into req and res objects that can be passed to middleware and route handlers. This makes it easier for developers to work with HTTP requests and responses without worrying about their underlying implementation.
Request and Response objects: The req and res objects in Express.js are data abstractions that provide a simplified interface for working with HTTP requests and responses. They encapsulate the complexity of working with raw HTTP messages and provide methods and properties for easy manipulation.
```js
app.get('/example', (req, res) => {
  const userAgent = req.get('User-Agent');
  const queryParams = req.query;
  res.send(`Hello, your user agent is: ${userAgent}, query params: ${JSON.stringify(queryParams)}`);
});

```
In this example, the req.get() method is used to get the User-Agent header, and req.query is used to access the query parameters of the request. These abstractions simplify working with HTTP requests.
### Adapter
Adapter Example: Ghost's Storage Adapter
Ghost uses adapters for its storage system, allowing users to switch between different storage providers (e.g., local storage, Amazon S3, Google Cloud Storage).

Here's an example of a storage adapter for Ghost called ghost-storage-adapter-s3:
```js
const AWS = require('aws-sdk');
const BaseAdapter = require('ghost-storage-base');

class S3Adapter extends BaseAdapter {
    constructor(config) {
        super();
        AWS.config.update(config);
        this._client = new AWS.S3();
        ...
    }

    exists(fileName, targetDir) {
        ...
    }

    save(image, targetDir) {
        ...
    }

    serve() {
        ...
    }

    delete(fileName, targetDir) {
        ...
    }

    read(options) {
        ...
    }
}

module.exports = S3Adapter;
```

```js
function make_rat(n, d) {
  const g = gcd(n, d);
  return pair(n / g, d / g);
}

function numer(x) {
  return head(x);
}

function denom(x) {
  return tail(x);
}

function add_rat(x, y) {
  return make_rat(numer(x) * denom(y) + numer(y) * denom(x), denom(x) * denom(y));
}

function sub_rat(x, y) {
  return make_rat(numer(x) * denom(y) - numer(y) * denom(x), denom(x) * denom(y));
}

function mul_rat(x, y) {
  return make_rat(numer(x) * numer(y), denom(x) * denom(y));
}

function div_rat(x, y) {
  return make_rat(numer(x) * denom(y), denom(x) * numer(y));
}

function equal_rat(x, y) {
  return numer(x) * denom(y) === numer(y) * denom(x);
}

function print_rat(x) {
  display(numer(x));
  display("/");
  display(denom(x));
}
```
