---
title: TypeScript Function Overloads
date: 2023-07-03 09:42:34
tags: TypeScript
---

Today I've come across something that seems baffling in Vuetify's [source code, genericComponent](https://github.com/vuetifyjs/vuetify/blob/master/packages/vuetify/src/util/defineComponent.tsx#L252): **a function with the same name declared multiple times**.

```ts
// No argument - simple default slot
export function genericComponent (exposeDefaults?: boolean): DefineComponentWithSlots<{ default: never }>

// Generic constructor argument - generic props and slots
export function genericComponent<T extends (new (props: Record<string, any>, slots: any) => {
  $props?: Record<string, any>
})> (exposeDefaults?: boolean): DefineComponentWithGenericProps<T>

// Slots argument - simple slots
export function genericComponent<
  Slots extends RawSlots
> (exposeDefaults?: boolean): DefineComponentWithSlots<Slots>

// Implementation
export function genericComponent (exposeDefaults = true) {
  return (options: any) => ((exposeDefaults ? defineComponent : _defineComponent) as any)(options)
}
```

At first glance, it might be perplexing. Why would the same function be declared four times? Furthermore, the first three declarations don't seem to be full function definitions at all, but rather TypeScript type declarations.

With this question in mind, let's look at an example from TypeScript official website.
```ts
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d);
  } else {
    return new Date(mOrTimestamp);
  }
}

const d1 = makeDate(12345678);
const d2 = makeDate(5, 5, 5);
const d3 = makeDate(1, 3);  // Error: No overload expects 2 arguments
```


Looking at the above code, you'll notice a function, makeDate, which can be invoked in two distinct ways. You might be tempted to ask, "Why not just use the third function? It appears to handle all possible scenarios, right?" 

From a purely technical perspective, you wouldn't be wrong. However, this is where we introduce an important concept in TypeScript: Function Overloading. With the first two function overloads, we're enhancing the development experience when calling makeDate by leaps and bounds.

![Screenshot 2023-07-29 at 19 30 35](https://github.com/flaming-cl/flaming-cl.github.io/assets/51183663/c53243a2-d914-43c5-ab91-4c7dbeb81301)

![Screenshot 2023-07-29 at 19 30 52](https://github.com/flaming-cl/flaming-cl.github.io/assets/51183663/b9fc33cb-fd0d-45ac-9504-5121c24c62a8)

![Screenshot 2023-07-29 at 19 31 18](https://github.com/flaming-cl/flaming-cl.github.io/assets/51183663/a35e3d00-654f-4c7f-97fc-6d67c06770cb)

With the above screenshots, you must have a much clearer idea about why we use function overloading. Yes, when a developer invokes `makeDate` with a single argument, TypeScript knows that this argument is a timestamp (defined in the first overload). And it won't prompt the developer for the additional parameters d and y. **In essence, overloads provide enhanced type checking and better autocompletion support in editors.** 

However, there's a critical rule to remember: the function implementation (e.g., the third `makeDate` declaration) must be compatible with all the overloads. That is, the parameters and return type of the implementation should be a superset of those in each overload. If we fail to do this, TypeScript will helpfully point out a mismatch. For example:

```ts
function fn(x: boolean): void;
function fn(x: string): void;  // Error: This overload signature is not compatible with its implementation signature.
function fn(x: boolean) {}
```

So, circling back to the genericComponent code, it's now clear that the initial 3 genericComponent functions are merely type definitions, outlining the possible outputs of genericComponent under different circumstances (i.e., DefineComponentWithSlots<{ default: never }>, DefineComponentWithSlots<Slots>, and DefineComponentWithGenericProps<T>).

In summary, while function overloading in TypeScript might initially appear perplexing, it is a useful feature that can greatly improve the developer experience by providing enhanced type safety and clearer code autocompletion suggestions. 
