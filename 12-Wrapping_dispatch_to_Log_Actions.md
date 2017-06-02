# 12. Wrapping `dispatch()` to Log Actions
[Video Link](https://egghead.io/lessons/javascript-redux-wrapping-dispatch-to-log-actions)

Now that our state shape is more complex, we want to override the `store.dispatch` function to add some `console.log()` statements so we can see how the state is affected by the actions.

We'll start by creating a new function called `addLoggingToDispatch` that accepts `store` as an argument. It's going to wrap the `dispatch` provided by Redux, so it reads the raw dispatch from `store.dispatch`.

`addLoggingToDispatch()` will return another function with the same signature as dispatch, which is a single action argument. Some browsers like Chrome support using `console.group()` to group several log statements under a single title, and we're passing in `action.type` in order to group several logs under the action type.

We will log the previous `state` before dispatching the action by calling `store.getState()`. Next, we will log the action itself so that we can see which action causes the change.

To preserve the contract of the `dispatch` function exactly, we'll declare `returnValue` and call the `rawDispatch()` function with the action. Now the calling code will not be able to distinguish between our function and the function provided by Redux, except that we are also logging some information.

We log the next state as well, because the store is guaranteed to be updated after the dispatch is called. We can use `store.getState()` in order to receive the next state after the dispatch.

#### `configureStore.js`
```javascript
const addLoggingToDispatch = (store) => {
  const rawDispatch = store.dispatch;
  return (action) => {
    console.group(action.type);
    console.log('prev state', store.getState());
    console.log('action', action);
    const returnValue = rawDispatch(action);
    console.log('next state', store.getState());
    console.groupEnd(action.type);
    return returnValue;
  }
}

const configureStore = () => {
  const persistedState = loadState();
  const store = createStore(todoApp, persistedState);

  store.dispatch = addLoggingToDispatch(store);
  .
  .
  .
```


### Finishing Touches

Chrome offers an API to style `console.log()`s, so we can add some colors to our logs. These modifications paint the previous state in gray, the action in blue, and the next state in green.

```javascript
console.log('%c prev state', 'color: gray', store.getState());
console.log('%c action', 'color: blue', action);
const returnValue = rawDispatch(action);
console.log('%c next state', 'color: green', store.getState());
```

If the `console.group()` API is not available in all browsers, we just return the raw dispatch as is.

```javascript
// At the top of `addLoggingToDispatch()` after `rawDispatch` is declared
if (!console.group) {
  return rawDispatch
}
```

Since it's not a good idea to log everything in production, we add a gate inside of `configureStore()` saying that if `process.env.NODE_ENV` is not production, then we're going to run this code. Otherwise, we're just going to leave the store as is.

#### Inside `configureStore`
```javascript
const configureStore = () => {
    const persistedState = loadState();
    const store = createStore(todoApp, persistedState);

    if (process.env.NODE_ENV !== 'production') {
      store.dispatch = addLoggingToDispatch(store);
    }
    .
    .
    .
```

Logging the action we dispatch along with the state before and after dispatching it makes it easy to find any mistakes.

[Recap at 2:18 in video](https://egghead.io/lessons/javascript-redux-wrapping-dispatch-to-log-actions)


<p align="center">
<a href="./11-Normalizing_the_State_Shape.md"><- Prev</a>
<a href="./13-Adding_a_Fake_Backend_to_the_Project.md">Next -></a>
</p>