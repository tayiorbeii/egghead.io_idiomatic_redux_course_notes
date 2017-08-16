# 18. Applying Redux Middleware
[Video Link](https://egghead.io/lessons/javascript-redux-applying-redux-middleware)

In the previous lesson, we figured out the contract that we want to use for our middleware functions.

However, middleware wouldn't be very useful if everybody had to implement `wrapDispatchWithMiddlewares` on their own.

Now we'll remove it, and instead import a utility called `applyMiddleware` from Redux.

#### `configureStore` Before:
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

Looking at our `configureStore` function, notice that we don't need the `store` right away. We can move the creation of the `store` down below where we specify the middleware.

We can also remove our custom `wrapDispatchWithMiddlewares` function, and instead we will `createStore` with the middleware right away.

The second argument to create store will be the result of calling `applyMiddleware` with my middleware functions as positional arguments.

This last argument to `createStore` is called an enhancer, and it's optional. If you want to specify the `persistedState`, you need to do this before the enhancer (you can also skip the `persistedState` if you don't have it).

#### `configureStore` After:
```javascript
// We also deleted `wrapDispatchWithMiddlewares()`

const configureStore = () => {
  const middlewares = [promise];
  if (process.env.NODE_ENV !== 'production') {
    middlewares.push(createLogger());
  }

  return createStore(
    todoApp,
    applyMiddleware(...middlewares)
  );
};
```

### Replacing our Custom Middleware

Many middlewares are available as npm packages. Both the `promise` and the `logger` middlewares that we wrote are no exceptions to this.

Running `npm install --save redux-promise` in a terminal will install a middleware that implements Promise support.

You can install a package called `redux-logger` in the same way, which is similar to the logger middleware we wrote before, but it's more configurable.

Inside of `configureStore.js`, we can now import and use our new middleware. Since we don't need to reference the `store` anymore, we can just return it directly from `configureStore`.

#### Updated `configureStore.js`
```javascript
import { createStore, applyMiddleware } from 'redux';
import promise from 'redux-promise';
import createLogger from 'redux-logger';
import todoApp from './reducers';

const configureStore = () => {
  const middlewares = [promise];
  if (process.env.NODE_ENV !== 'production') {
    middlewares.push(createLogger());
    // Note: you can supply options to `createLogger()`
  }

  return createStore(
    todoApp,
    applyMiddleware(...middlewares)
  );
};

export default configureStore;
```

[Recap at 2:06 in video](https://egghead.io/lessons/javascript-redux-applying-redux-middleware)


<p align="center">
<a href="./17-The_Middleware_Chain.md"><- Prev</a>
<a href="./19-Updating_the_State_with_the_Fetched_Data.md">Next -></a>
</p>