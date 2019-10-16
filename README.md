# :tophat: Classy State

> Don't fear the stateful classes.

1. Write easy-to-grasp class with methods that mutate and access the state.
2. Run it with immutable backend powered by [Immer](https://immerjs.github.io/immer/docs/introduction).
3. ???
4. Profit!

## :warning: Work in progress

This repo does not contain any code other then example of how *Classy State* might be used and explanation of how it is achieved.

If you want to try `Classy State` as it is currently implemented, go to this CodeSandbox – [Classy State Playground](https://codesandbox.io/s/new-kwwhp).

## Example

Let's say you want a canonical state managment example – a counter. Without any knowledge about React, Redux, etc. you might write following:

```js
// Counter.js

// plain old class with mutable state
export class Counter {
  state = 0
  increase() {
    this.state += 1;
  }
  decrease() {
    this.state -= 1;
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
const counter = new Counter();
console.log(counter.state); // 0
console.log(counter.isValid()) // true

counter.increase();
console.log(counter.state); // 1

counter.increase();
console.log(counter.state); // 2

counter.reset();
console.log(counter.state); // 0

counter.decrease();
console.log(counter.isValid()); // false
```

This is considered **a bad code** by the React community because:

1. using mutable state is a bad practise
2. you can't use it in your React component

These are valid objections that should be adressed.

First objection, mutability of the state, is beatifully solved by [Immer](https://immerjs.github.io/immer/docs/introduction). With Immer you can write seemingly mutable code that won't change the original state. It will produce a new state that is structurally shared with the original one – immutability is preserved.

Second objection can be distilled to the following: React doesn't know when the state has changed and so it doesn't know when to rerender. This is where *Classy State* comes in...

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

`useClassyState` takes definition of your stateful class and transforms it into an object that behaves as you (and React) would expect. On top of that, `counter` (the transformed object) is typed as an instance of the `Counter` class, so you get proper code-completion and type-checking for free.

## Magic

Method on an instance of our class, i.e. `(new Counter()).increase`, is a function with the instance bound to `this` keyword. This function lives on the class prototype, i.e. `Counter.prototype.increase`. We can take this function from the prototype and bind them to anything we wish via `Function.prototype.bind` function – `Counter.prototype.increase.bind({ state: 0 })()`.

So what if we take definition of a method, e.g. `increase`, and augment it with Immer's produce function:

```js
import { produce } from "immer";

const increase = Counter.prototype.increase

const immutableIncrease = produce(
  function(draft) {
    increase.apply(draft)()
  }
);

const instance = {
  state: 0
};

const augmentedInstance = immutableIncrease(instance);
```

So what have we done is basically took a function that mutates class instance, and we transformed it into a non-mutating function that produces new instance.

This is the main insight of `Classy State`, everything else is just an implementation detail.

## FAQ

### Do you hate Redux?

No, but *stores*, *reducers*, *actions*, *dispatch*, *multi-dispatch*, *switch-case*, *action types*, *action creators*, *async action creators*, *middleware*, *thunks*, *sagas*, etc. are a bit too much.
