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
}
```

And you would use it like this:

```
const counter = new Counter();
counter.increase();
counter.decrease();
counter.reset();
```

This is considered **a bad code** by the React community because:

1. using mutable state is considered a bad practise
2. you can't use it in your React component

These are valid objections that should be adressed.

First objection, mutability of the state, is beatifully solved by [Immer](https://immerjs.github.io/immer/docs/introduction). With Immer you can write seemingly mutable code that won't change the original state. It will produce a new state that is structurally shared with the original one, so the immutability is preserved.

Second objection can be distilled to the following: React doesn't know when the state has changed and so it doesn't know when to rerender. This is where *Classy State* comes in...

```js
// CounterApp.js

import React from "react";
import { useClassyState } from "classy-state";

// Import your stateful class...
import { Counter } from "./Counter";

function CounterApp() {

  // This is where magic happens...
  const counter = useClassyState(Counter);
  
  return (
    <div>
      <div>{counter.state}</div>
      <button onClick={counter.inc}>Inc</button>
      <button onClick={counter.dec}>Dec</button>
      <button onClick={counter.reset}>Reset</button>
    </div>
  );
}
```

This is how you should use your stateful class inside React code.

`useClassyState` takes definition of your stateful class and transforms it into an object that behaves as React would expect. On top of that, `counter` (the transformed object) is typed as an instance of `Counter` class, so you get proper code-completion and type-checking for free.

## Magic

Methods in classes are functions that have an object that is an instance of a class, bound to the `this` keyword. This binding can be done by `instance.method()` invocation or you can explicitly bind an object (and parameters) with `Function.prototype.bind` function – `fn.bind(instance)()`.

So what if we take definition of a method, e.g. `increase`, and augment it with Immer's produce function:

```js
import { produce } from "immer";

// taken out from the class definition
const increase = function() {
  this.state += 1;
};

const immutableIncrease = produce(
  function(draft) {
    increase.bind(draft)()
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
