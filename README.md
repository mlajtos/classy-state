# :tophat: Classy State

> Don't fear the stateful classes.

1. Write easy-to-grasp class with methods that mutate and access the state.
2. Run it with immutable backend powered by [Immer](https://immerjs.github.io/immer/docs/introduction).
3. ???
4. Profit!

## :warning: Work in progress

This repo does not contain any code other then example of what *Classy State* is, how it might be used, and an explanation of how it works.

If you want to try *Classy State* as it is currently implemented, go to this CodeSandbox – [Classy State Playground](https://codesandbox.io/s/new-kwwhp).

## Guide

### State without ceremony

Let's say you want a canonical state managment example – a counter. Without any knowledge about React, Redux, etc. you might write the following:

```js
// Counter.js

// plain old class with mutable state
export class Counter {
  state = 0
  increase(step = 1) {
    this.state += step;
  }
  decrease(step = 1) {
    this.state -= step;
  }
  reset() {
    this.state = 0;
  }
  isValid() {
    return this.state >= 0;
  }
}
```

And you would use it like this:

```
import { Counter } from "Counter";

const counter = new Counter();
console.log(counter.state); // 0
console.log(counter.isValid()) // true

counter.increase();
console.log(counter.state); // 1

counter.increase(10);
console.log(counter.state); // 11

counter.reset();
console.log(counter.state); // 0

counter.decrease(3);
console.log(counter.isValid()); // false
```

This is a bit boring, but valid and straightforward code. However by the React community it is considered to be **a bad code** because:

1. using mutable state is a bad practise
2. you can't use it in your React component

These are valid objections that should be adressed.

### Mutable state

The first objection, im/mutability of the state, is beatifully solved by [Immer](https://immerjs.github.io/immer/docs/introduction). Immer lets you write seemingly mutable code that won't change the original state. It will produce a new state that is structurally shared with the original one – immutability is preserved.

Immer is the essential part of *Classy State*, however you as the user will never know about it. So let's sweep this problem under the carpet for a while and we will pretend mutable code is okay.

### Reactivity

Second objection can be distilled to the following: React doesn't know when the state has changed and so it doesn't know when to rerender. Reactivity, as we might call this issue, is related, but not synonymous to the immutability of the state. React solves this issue with explicit calls to functions that mutates the state. Now, when we touched React a bit, it might be good time to use `Counter` in our app:

```js
// CounterApp.js

import React from "react";
import { useClassyState } from "classy-state";

// Import your stateful class...
import { Counter } from "./Counter";

function CounterApp() {

  // This is where magic happens.
  const counter = useClassyState(Counter);
  
  // Now you can use `counter` as you would expect...
  return (
    <>
      <div style={{ color: counter.isValid() ? "green" : "red" }}>
        {counter.state}
      </div>
      <button onClick={counter.increase}>
        Increase
      </button>
      <button onClick={counter.decrease}>
        Decrease
      </button>
      <button onClick={counter.reset}>
        Reset
      </button>
    </>
  );
}
```

Again, this is pretty straightforward and a bit boring code. However, there is the magic call to `useClassyState` that makes everything work. If you would create a new instance of the `Counter` – `const counter = new Counter()` – and used it instead of the hook call, you would be surprised because your app wouldn't work.

What `useClassyState` hook does is it takes the definition of your stateful class and transforms it into an object that behaves as you (and React) would expect. On top of that, `counter` (the transformed object) is typed as an instance of the `Counter` class, so you get proper code completion and type checking for free.

From the user perspective, this is all there is to *Classy State*. There is just a single hook and no additional concepts you need to learn – no stores, no reducers, no actions, no dispatch, no multi-dispatch, no switch-case, no action types, no action creators, no async action creators, no middleware, no thunks, no sagas, etc. This won't replace Redux, but [you might not need Redux](https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367) anyway.

### Under the hood

So how does `useClassyState` works? This is a step-by-step guide of how it might be implemented... This explanation describes naive implementation, but is good way to grasp the inner workings. [Current implementation](https://codesandbox.io/s/new-kwwhp) uses Proxies, has bugs and is cheating a bit – you have been warned.

### Classes + methods === prototypes + functions

Method on an instance of a class, i.e. `(new Counter()).increase`, is a function with the instance bound to `this` keyword. This function lives on the class prototype, i.e. `Counter.prototype.increase`. We can take this function from the prototype and apply/call it with any object we wish – `Counter.prototype.increase.apply({ state: 0 })`.

So what if we take definition of a method, e.g. `increase`, and augment it with Immer's produce function:

```js
import { produce } from "immer";

const increase = produce(
  (draft) => {
    Counter.prototype.increase.apply(draft);
  }
);

const instance = {
  state: 0
};

const nextInstance = increase(instance);
console.log(instance, nextInstance); // { state: 0 }, { state: 1 }
```

What have we done is basically took a function that mutates class instance, and we transformed it into a non-mutating function that produces new instance. That is really convenient and super easy thanks to Immer.

### Arguments

Our `Counter.prototype.increase` takes an optional argument that we do not take into consideration, so let's change that:


```js
import { produce } from "immer";

const increase = produce(
  (draft, ...args) => {
    Counter.prototype.increase.apply(draft, args);
  }
);

const instance = {
  state: 0
};

const nextInstance = increase(instance, 3);
console.log(instance, nextInstance); // { state: 0 }, { state: 3 }
```

Easy.

### Return values

Our method augmentation won't work for `Counter.prototype.isValid` because it needs to return a value based on the state. After small refactoring, we might get something like this:

```js
import { produce } from "immer";

const isValid = (instance, ...args) => {
  let returnValue = undefined;
  
  const nextInstance = produce(
    instance,
    draft => {
      returnValue = Counter.prototype.isValid.apply(draft, args);
    }
  );
  
  return [nextInstance, returnValue];
};

const instance = {
  state: 0
};

const [nextInstance, returnValue] = isValid(instance);
console.log(instance, nextInstance, returnValue); // { state: 0 }, { state: 0 }, true
```

### Method-producing function

We can abstract away the concrete function we've been using so far and create a function that will create our desired immutable method:

```js
import { produce } from "immer";

const createMethod = (fn) => (instance, ...args) => {
  let returnValue = undefined;
  
  const nextInstance = produce(
    instance,
    draft => {
      returnValue = fn.apply(draft, args);
    }
  );
  
  return [nextInstance, returnValue];
};

const increase = createMethod(Counter.prototype.increase); 
const isValid = createMethod(Counter.prototype.isValid);

const instance_0 = {
  state: 0
};

const [instance_1, returnValue_1] = increase(instance_0, 2);
console.log(instance_1, returnValue_1); // { state: 2 }, undefined
const [instance_2, returnValue_2] = isValid(instance_1);
console.log(instance_2, returnValue_2); // { state: 2 }, true
console.log(instance_1 === instance_2); // true
```

Last line shows us something very important – we can distinguish between methods that are mutating the instance and methods that are "pure". This is very crucial distinction that will be later important.

### Mutations

In previous step we used `instance_0`, `instance_1`, etc. to distinguish instance at different point in time. Let's refactor it a bit:

```js
const createMethod = (fn) => (instance, ...args) => {
  // same as before
};

const increase = createMethod(Counter.prototype.increase); 
const decrease = createMethod(Counter.prototype.decrease);
const reset = createMethod(Counter.prototype.reset);

// This is like React.useState, but
// accessor and mutator are both functions.
// And there is an extra value that tracks the history.
const useFakeState = (initialState) => {
  const states = [initialState];
  const currentState = () => states[states.length - 1];
  const setState = (newState) => {
    states.push(newState);
  };
  return [currentState, setState, states];
}

const [instance, setInstance, instances] = useFakeState({ state: 0 });

setInstance(increase(instance())[0]);
setInstance(increase(instance())[0]);
setInstance(reset(instance())[0]);
setInstance(decrease(instance())[0]);

console.log(instances);

/*

[
  {state: 0},
  {state: 1},
  {state: 2},
  {state: 0},
  {state: -1}
]

*/
```

This may remind you of something...

### All the methods

We don't know in advance what methods will be on the stateful class so we need to transform all of them:

```js
const createMethod = (fn) => (instance, ...args) => {
  // same as before
};

const createMethods = StatefulClass => (
  Object.fromEntries(
    Object.getOwnPropertyNames(StatefulClass.prototype).map(methodName => [
      methodName,
      method(StatefulClass.prototype[methodName])
    ])
  )
);

const methods = createMethods(Counter);

const [instance, setInstance, instances] = useFakeState({ state: 0 });

setInstance(methods.increase(instance())[0]);
setInstance(methods.increase(instance())[0]);
setInstance(methods.reset(instance())[0]);
setInstance(methods.decrease(instance())[0]);

```

### Wrapping it all

**TODO: methods object should contain all the logic for mutation**

```js
const createMethod = (fn) => (instance, ...args) => {
  // same as before
};

const createMethods = StatefulClass => (
  Object.fromEntries(
    Object.getOwnPropertyNames(StatefulClass.prototype).map(methodName => [
      methodName,
      method(StatefulClass.prototype[methodName])
    ])
  )
);

const method = (functionFromPrototype) => (instance) => (...args) => {
  const fn = createMethod(functionFromPrototype);
  const [instance, returnValue] = fn(state, ...args);
}


const methods = createMethods(Counter);

const [instance, setInstance, instances] = useFakeState({ state: 0 });

setInstance(methods.increase(instance())[0]);
setInstance(methods.increase(instance())[0]);
setInstance(methods.reset(instance())[0]);
setInstance(methods.decrease(instance())[0]);

```

---

*To be continued...*
