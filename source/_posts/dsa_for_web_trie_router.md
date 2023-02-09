---
title: Trie based router
date: 2022-12-21 13:04:15
tag: DSA for web dev
---
## Why trie based router
If you do not have performance issues, adopting a trie based router might become a premature optimization. In this case, Regex based routing is fast enough, as Regex are compiled down to native code in modern browsers, like Chrome.

However, if you feel tradition MVC routing results in performance pressure on your web App, you can take a look at trie based router. Here is an [example](https://ayende.com/blog/173313/trie-based-routing) from Ayende Rahien.   
> Traditional MVC routing infrastructure can consume a significant amount of time,
leaving little time for handling actual requests.
By using a trie based router, it is possible to reduce time spent on routing infrastructure,
and increase time spent on actual request handling.

## How to implement Router trie
A route mainly does two things: registering a route with a function; matching a route.
Here we use trie to register and match routes.
### 1. Register a route
First, split the route into an array of routes.
e.g. "/courses/:id" => ["courses", ":id"]
```javascript
const add = (route) => {
    if (typeof route !== 'string') {
        throw TypeError('route should be string');
    }
    this.currentRoutes = splitRoute(route);
    return this.createTrieNode(0, this.trie);
}
```
Next, recursively add each segment of the route array to our trie.
- 1/ Handle ":" for dynamic values, "*" for wildcard (any value)
- 2/ If a segment is found in our trie, check the next one
  - "courses" is shared in the trie, if we have both courses/:id and courses/ranking 
- 3/ If not found, create a trie node for this segment 
```javascript
const createTrieNode = (index, trie) => {
    const thisRoute = this.currentRoutes[index];
    if (thisRoute === undefined) return trie;
        
    let node = null;
    if (HAS_SPECIAL_SYMBOL) {
        handleSpecialSymbol();
    } else if (trie.nodes.has(thisRoute)) {
        node = trie.nodes[thisRoute];
    } else {
        node = this.createNewNodes();
        trie.nodes[thisRoute] = node;     
    }

    return this.createTrieNode(index + 1, node);
}
```
### 2. Match a route
Logics for match() a route is quite similar to add().
The main difference is in match() we also handle params for a route (if found).
```javascript
const match = (route) => {
    this.currentRoutes = splitRoute(route);
    this.currentParams = {};
        
    const node = search(0, this.trie);
    if (!node) return undefined;
        
    node = new Map(node);
    node.params = this.currentParams;
    return node;
}
```
What does search() do?
If a node is found, search() recursively calls itself, with the next segment and the matching node as arguments.
```javascript
const search = (index, trie) => {
    if (trie === undefined) return trie;
    const thisRoute = this.currentRoutes[index];
    if (thisRoute === undefined) return trie;

    if (trie.nodes.has(thisRoute)) {
        return search(index + 1, trie.nodes[thisRoute]);
    }

    if (trie.name) {
        // add params
    }

    if (trie.wildCard) {
        // add params
    }

    return search(index + 1);
}
```
For full implementation, check [here](https://github.com/choojs/wayfarer/blob/master/trie.js).

### References

[1] [Trie based routing, Ayende Rahien](https://ayende.com/blog/173313/trie-based-routing)
[2] [A composable trie based router, Wayfarer](https://github.com/choojs/wayfarer/blob/master/trie.js)
[3] [Trie based routing instead of Regex based routing, React router](https://github.com/remix-run/react-router/issues/6627)
