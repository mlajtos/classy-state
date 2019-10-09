# 🎩 Classy State

Let's face it – writing reducers with all the orchestration is pain in the ass. On the other hand, writing shared mutable state is a cardinal sin. Imperative code that mutates the state feels straightforward and easy. So. Much. Headache. Is there a middle path? Now there is – `useClassyState` hook.

See [classy-state-playground CodeSandbox](https://codesandbox.io/s/new-kwwhp) for live demo.

## Example

```js
// legacy-code.js

export class Counter {
  state = 0
  inc() {
    this.state += 1;
  }
  dec() {
    this.state -= 1;
  }
  reset() {
    this.state = 0;
  }
  resetWithDelay() {
    setTimeout(
      () => this.reset(),
      1000
    )
  }
}
```

```js
// ui.js

import React from "react";
import { useClassyState } from "classy-state";

import { Counter } from "./legacy-code";

function CounterUI() {
  const counter = useClassyState(Counter);
  
  return (
    <div>
      <div>{counter.state}</div>
        <button onClick={counter.inc}>Inc</button>
        <button onClick={counter.dec}>Dec</button>
        <button onClick={counter.reset}>Reset</button>
        <button onClick={counter.resetWithDelay}>Reset with delay</button>
    </div>
  );
}
```

## WAT

Yes.

## How?

Proxies and Immer.

## But why?

Because why not.
