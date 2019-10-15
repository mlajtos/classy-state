# :tophat: Classy State

> Don't fear the stateful classes.

1. Write easy-to-grasp class with methods that mutate and access the state.
2. Run it with immutable backend powered by [Immer](https://immerjs.github.io/immer/docs/introduction).
3. ???
4. Profit!

## :warning: Work in progress

This repo does not contain any code other then examples of how *Classy State* might be used. If you want code, go and try this CodeSandbox – [Classy State Playground](https://codesandbox.io/s/new-kwwhp).

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

## FAQ

### Do you hate Redux?

No, but *stores*, *reducers*, *actions*, *dispatch*, *multi-dispatch*, *switch-case*, *action types*, *action creators*, *async action creators*, *middleware*, *thunks*, *sagas*, etc. are a bit too much.
