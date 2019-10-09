# 🎩 Classy State

Here are the facts:

1. Writing good state managment code in reactive programming paradigm is pain in the ass.
2. Using shared mutable state is big no-no.
3. Imperative code with mutations feels straightforward and easy.

*So. Much. Headache.*

Is there a middle path?

Yes, [`useClassyState` hook](https://codesandbox.io/s/new-kwwhp).

## Example

```js
// Counter.js

// plain old class with mutable state
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
// CounterApp.js

import React from "react";
import { useClassyState } from "classy-state";

import { Counter } from "./Counter";

function CounterApp() {

  // magic object with immutable state
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

## FAQ

### WAT

Yes, Javascript is awesome.

### How does it work?

[Proxies](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) and [Immer](https://immerjs.github.io/immer/docs/introduction). Magic on top of more magic.

### But why?

Because *stores*, *reducers*, *actions*, *dispatch*, *multi-dispatch*, *switch-case*, *action types*, *action creators*, *async action creators*, *middleware*, *thunks*, etc. are a bit too much.

### Is this a joke?

Yes and no. [The code](https://codesandbox.io/s/new-kwwhp) works, but it's a hacky proof-of-concept. You can play with it and will probably break it easily.

It seems weird not to share such a cool hack.

---

<sub><sup>Stay classy. ❤️</sup></sub>
