---
title: Exploring the Benefits of Locality in Memory
date: 2022-11-10 13:04:15
---
As stated in Computer Systems: A Programmer's Perspective:
> Well-written computer programs tend to exhibit good locality

## What is locality
<details><summary><b>If you haven't heard of locality, click here.</b></summary>
<div>
 
> Locality is often described as temporal locality and spatial locality.

## 1. Spatial locality
Good practice: reference a memory location and then its neighbors
### Poor spatial locality
```c
int sumarraycols (int a[M][N]) {
  int i, j, sum = 0;
  for (j = 0; j < N; j++)
    for (i =0: i < M; i++)
      sum+ = a[i][j]:
  return sum;
}
```
This code jumping through elements of different rows in the matrix.

Assume you are going to buy groceries and books. With poor spatial locality, you  do this:
- buy tomatoes first, and go to a book store for a Math book.
- back to the grocery store for milk, and go to the book store again for a comic book.

### Better spatial locality
```c
int sumarraycols (int a[M][N]) {
  int i, j, sum = 0;
  for (i = 0; i < M; i++)
    for (j =0: j < N; j++)
      sum+ = a[i][j]:
  return sum;
}
```
This code reads all the elements in a row and then the next row.

Back to our grocery example, this time you buy everything from the grocery store first. 
Later to buy all the books you need from the book store.

## 2. Temporal locality
Good practice: reference a memory location multiple times
### Good temporal locality
Our code above repeatedly reference the same variables (i, j) and enjoy good temporal locality.

</div>
</details>

## Example in JavaScript
Can we have good spatial locality if we loop cols before rows like this?
```javascript
function farAccess(data) {
  for (let i = 0; i < COLS; i++) {
    for (let j = 0; j < ROWS; j++) {
      data[j * ROWS + i].x = 1;
    }
  }
}
```
Let's look at an [example](https://codesandbox.io/s/cool-murdock-bnj02z?file=/src/index.js) given by Yonatan Kra:

1/ Create an array of 1,000,000 instances.
```javascript
const ROWS = 1000, COLS = 1000, REPEATS = 1000;
const testArray = new Array(ROWS * COLS).fill(0).map((a, i) => new Boom(i));

// testArray:
0:Boom {id: 0, x: 0}
1:Boom {id: 1, x: 0}
2:Boom {id: 2, x: 0}
3:Boom {id: 3, x: 0}
...
999999:Boom {id: 999999, x: 0}
```
2/ Create a tricky array.
```javascript
const trickyArray = new Array(ROWS * COLS).fill(0);
for (let col = 0; col < COLS; col++) {
    for (let row = 0; row < ROWS; row++) {
        trickyArray[row * ROWS + col] = arr[col * COLS + row];
    }
}
```
3/ Write two functions with good/bad spatial locality. 
```javascript
function runGoodSpatialLocality(arr) {
  for (let i = 0; i < ROWS; i++) {
    for (let j = 0; j < COLS; j++) {
      arr[i * ROWS + j].x = 0;
    }
  }
}

function runPoorSpatialLocality(arr) {
  for (let i = 0; i < COLS; i++) {
    for (let j = 0; j < ROWS; j++) {
      arr[j * ROWS + i].x = 0;
    }
  }
}
```
4/ Test performance
```javascript
function timeit(cb, type) {
    console.log(`Started data ${type}`);
    const start = performance.now();
    for (let i = 0; i < repeats; i++) {
        cb();
    }
    const end = performance.now();
    console.log(
        `Finished data locality test run in ${((end - start) / 1000).toFixed(
            4
        )} seconds`
    );
    return end - start;
}

timeit(() => runGoodSpatialLocality(testArray), "good");
setTimeout(() => {
    timeit(() => runPoorSpatialLocality(testArray), "bad");
    setTimeout(() => {
        timeit(() => runPoorSpatialLocality(trickyArray), "tricky");
    }, 2000);
}, 2000);
```
5/ Results
The second function call with bad locality is more than 2 times slower.
This is because it makes the CPU chasing its tail to jump between N (length of a row) stride of memory, during each access.
But why the third function call with bad locality and the tricky array is still fast? 
```javascript
Finished good data locality test in 5.1799 seconds
Finished bad data locality test in 13.9438 seconds
Finished tricky data locality test in 5.4242 seconds
```
Because our tricky array still access 1-[stride](https://inst.eecs.berkeley.edu/~cs61c/sp08/labs/12/) of an array:
Although we access the whole array by index 0, 1000, ... 
our tricky array just put elements with 1-stride of memory on index 0, 1000, 2000, ... like this:
```javascript
// trickArray:
0:Boom {id: 0, x: 0}
1:Boom {id: 1000, x: 0}
1000:Boom {id: 1, x: 0}
1001:Boom {id: 1001, x: 0}
2000:Boom {id: 2, x: 0}
...

// testArray:
0:Boom {id: 0, x: 0}
1:Boom {id: 1, x: 0}
1000:Boom {id: 1000, x: 0}
2000:Boom {id: 1000, x: 0}
...
```
Now we get the answer: Yes. 
Even though we run a COL loop first and a ROW loop later, 
as we still access a memory location and its neighbors in memory,
we can have good spatial locality.

### References

[1] [Computer Systems: A Programmer's Perspective](https://csapp.cs.cmu.edu)
[2] [Memory in Javascript—Beyond Leaks](https://medium.com/walkme-engineering/memory-in-javascript-beyond-leaks-8c1d697c655c)
[3] [Stride, Caches, CS61C lab12](https://inst.eecs.berkeley.edu/~cs61c/sp08/labs/12/)
