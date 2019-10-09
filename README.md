# 🎩 Classy State

You are tasked with coding a brand new React fronted for some legacy code that uses shared mutable state inside a class. Don't kill yourself just yet, try `useClassyState` hook first.

## Example

See [classy-state-playground Codesandox](https://codesandbox.io/s/new-kwwhp) for live demo.

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
        <button onClick={counter.dec}>Inc</button>
        <button onClick={counter.reset}>Reset</button>
    </div>
  );
}
```

## Instalation

🚫There is no NPM package and won't be.

## WAT

Yes.

## How?

Proxies and Immer.

## But why?

Why not.
