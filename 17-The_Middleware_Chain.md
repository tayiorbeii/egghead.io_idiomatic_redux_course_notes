# 17. The Middleware Chain
[Video Link](https://egghead.io/lessons/javascript-redux-the-middleware-chain)

In our last lesson, we wrote two functions that wrap the `dispatch` function to add custom behavior. Let's take a closer look at how they work together.

```javascript
const configureStore = () => {
  const store = createStore(todoApp);

  if (process.env.NODE_ENV !== 'production') {
    store.dispatch = addLoggingToDispatch(store);
  }

  store.dispatch = addPromiseSupportToDispatch(store);

  return store;
};
```

The final version of the `dispatch` function before returning the `store` is the result of calling `addPromiseSupportToDispatch`.

#### `addPromiseSupportToDispatch` Before:
```javascript
const addPromiseSupportToDispatch = (store) => {
  const rawDispatch = store.dispatch;
  return (action) => {
    if (typeof action.then === 'function') {
      return action.then(rawDispatch);
    }
    return rawDispatch(action);
  };
};
```

The function returned by `addPromiseSupportToDispatch` acts like a normal `dispatch` function, but if it gets a Promise it waits for it to resolve, and then passes the result to `rawDispatch` (where `rawDispatch` is the previous value of `store.dispatch`).

For non-promises, the function calls `rawDispatch` right away. `rawDispatch` corresponds to store.dispatch at the time `addPromiseSupportToDispatch` was called.


### Refactoring our `dispatch`-enhancing Functions

Since `store.dispatch` was reassigned earlier (inside of `configureStore`), it's not completely fair to refer to it as `rawDispatch` inside of `addPromiseSupportToDispatch`.

We'll rename `rawDispatch` to `next`, because this is the next `dispatch` function in the chain.

#### `addPromiseSupportToDispatch` After:
```javascript
const addPromiseSupportToDispatch = (store) => {
  const next = store.dispatch;
  return (action) => {
    if (typeof action.then === 'function') {
      return action.then(next);
    }
    return next(action);
  };
};
```

Above, `next` refers to the `store.dispatch` that was returned from `addLoggingToDispatch()`.

Recall that our `addLoggingToDispatch()` function also returns a function with the same API as the original `dispatch` function, but it logs the action `type`, the previous `state`, the `action`, and the next `state` along the way.

It calls `rawDispatch` which corresponds to `store.dispatch` at the time that  `addLoggingToDispatch` was called. In this case, this is the `store.dispatch` provided by `createStore()` inside of `configureStore`.

However, it is entirely conceivable that we might want to override the `dispatch` function before adding the logging.

For consistency, we will rename `rawDispatch` to `next` here as well. In this particular case, `next` points to the original `store.dispatch`.

#### `addLoggingToDispatch()`
```javascript
const addLoggingToDispatch = (store) => {
  const next = store.dispatch;
  if (!console.group) {
    return next;
  }

  return (action) => {
    console.group(action.type);
    console.log('%c prev state', 'color: gray', store.getState());
    console.log('%c action', 'color: blue', action);

    const returnValue = next(action);

    console.log('%c next state', 'color: green', store.getState());
    console.groupEnd(action.type);
    return returnValue;
  };
};
```

### Introducing Middleware Functions

While this method of extending the store works, it's not really great that we override the public API and replace it with custom functions.

To get away from this pattern, we will declare an array of _middleware functions_, which is just a fancy name for the extra-functionality functions we wrote.

This `middlewares` array will contain functions to be applied later as a single step.

We'll push `addLoggingToDispatch` and `addPromiseSupportToDispatch` to the middleware array.

Now we create a function `wrapDispatchWithMiddlewares()` that takes the `store` as the first argument, and the array of middlewares as the second.

#### Refactoring `configureStore`
```javascript
const configureStore = () => {
  const store = createStore(todoApp);
  const middlewares = [promise];

  if (process.env.NODE_ENV !== 'production') {
    middlewares.push(logger);
  }

  wrapDispatchWithMiddlewares(store, middlewares);

  return store;
};
```

Inside of `wrapDispatchWithMiddlewares()` we're going to use `middlewares`'s `forEach` method to run some code for every middleware.

Specifically, we will override the `store.dispatch` function to point to the result of calling the middleware with the `store` as an argument.

```javascript
const wrapDispatchWithMiddlewares = (store, middlewares) =>
  middlewares.forEach(middleware =>
    store.dispatch = middleware(store);
  );
```

Recall that inside of our middleware functions themselves, there is a certain pattern that we have to repeat. We grabbing the value of `store.dispatch` and store it in a variable called `next` that we call later.

To make it a part of the middleware contract, we can make `next` an outside argument, just like the `store` before it and the `action` after it.

#### Updating `addLoggingToDispatch()`
```javascript
const addLoggingToDispatch = (store) => {
  return (next) => {
    if (!console.group) {
      return next;
    }

    return (action) => {
      console.group(action.type);
      console.log('%c prev state', 'color: gray', store.getState());
      console.log('%c action', 'color: blue', action);

      const returnValue = next(action);

      console.log('%c next state', 'color: green', store.getState());
      console.groupEnd(action.type);
      return returnValue;
    };
  }
};
```

With this change, the middleware becomes a function that returns a function that returns a function.

This pattern is called **currying**. This is not very common in JavaScript, but is actually very common in functional programming languages.

#### Updating `addPromiseSupportToDispatch`:
```javascript
const addPromiseSupportToDispatch = (store) => {
  return (next) => {
    return (action) => {
      if (typeof action.then === 'function') {
        return action.then(next);
      }
      return next(action);
    };
  }
};
```

Again, rather than take the next middleware from the store, we will make it injectable as an argument so that the function that calls the middlewares can choose which middleware to pass.

Finally, since `store` is not the only injected argument, we also need to inject the next middleware, which is the previous value of `store.dispatch`.

#### Updating `wrapDispatchWithMiddlewares`
```javascript
const wrapDispatchWithMiddlewares = (store, middlewares) =>
  middlewares.slice().reverse().forEach(middleware => {
    store.dispatch = middleware(store)(store.dispatch);
  });
```

Now that middlewares are a first-class concept, we can rename `addLoggingToDispatch` to just `logger`, and rename `addPromiseSupportToDispatch` to `promise`.


### Arrow-ifying our `promise` Middleware

The curried style of function declaration can get very hard to read. Luckily we can use arrow functions and rely on the fact that they can have expressions as their bodies.

```javascript
// Before
const addPromiseSupportToDispatch = (store) => {
  return (next) => {
    return (action) => {
      if (typeof action.then === 'function') {
        return action.then(next);
      }
      return next(action);
    };
  }
};

// After
const promise = (store) => (next) => (action) => {
  if (typeof action.then === 'function') {
    return action.then(next);
  }
  return next(action);
}
```

It is still a function that returns a function returning a function, but it's much easier to read.

The mental model you can use for this is "_this is just a function with several arguments that are applied as they become available_".

### Wrapping up Middlewares

Our middlewares are currently specified in the order in which the `dispatch` function is overridden, but it would be more natural to specify the order in which the action propagates through the middlewares.

We will change our middleware declaration to specify them in the order in which the action travels through them:

```javascript
const configureStore = () => {
  const store = createStore(todoApp);
  const middlewares = [promise];
  .
  .
  .
```

We will also `wrapDispatchWithMiddlewares` from right to left by cloning the past array then reversing it.

```javascript
const wrapDispatchWithMiddlewares = (store, middlewares) =>
  middlewares.slice().reverse().forEach(middleware => {
    store.dispatch = middleware(store)(store.dispatch);
  });
```

[Recap at 5:42 in video](https://egghead.io/lessons/javascript-redux-the-middleware-chain)


<p align="center">
<a href="./16-Wrapping_dispatch_to_Recognize_Promises.md"><- Prev</a>
<a href="./18-Applying_Redux_Middleware.md">Next -></a>
</p>
