# 23. Avoiding Race Conditions with Thunks
[Video Link](https://egghead.io/lessons/javascript-redux-avoiding-race-conditions-with-thunks)

When we increase the delay in our fake API client to five seconds, we notice a problem. We aren't checking if the tab is already loading before starting a request, and then a bunch of `receiveTodos` actions come back, potentially resulting in a race condition.

To fix this, we can exit early from the `fetchTodos` action creator if we know that we are already fetching the todos for the given filter.

Inside of `fetchTodos`, we will add an `if` to check if we are currently fetching by using our `getIsFetching` selector that accepts the store state and the filter as arguments. If it returns `true`, we will exit early from the thunk without dispatching any actions.

#### Updating `fetchTodos`
```javascript
export const fetchTodos = (filter) => (dispatch) => {
  if (getIsFetching(getState(), filter)) {
    return;
  }
  /// rest of fetchTodos
```

The `getIsFetching` selector is defined inside the top level reducer file, so we need to import it as a named `import` from `reducers`.

```javascript
import { getIsFetching } from '../reducers';
```

We are also using `getState()` that isn't defined in this file. It belongs to the `store` object, but we don't have access to it directly from the action creator.

### Updating the `thunk` Middleware

We can make it so that the `thunk` middleware inside `configureStore.js` injects not just the `store.dispatch()` function inside the thunk actions, but also the `store.getState` function. This way, we can grab it as a second argument after `dispatch` inside of the `thunk` action creator.

```javascript
// Inside configureStore.js
const thunk = (store) => (next) => (action) =>
  typeof action === 'function' ?
    action(store.dispatch, store.getState) :
    next(action);
```

```javascript
// Add `getState` as a second parameter to `fetchTodos`
export const fetchTodos = (filter) => (dispatch, getState) => {
```

With these changes, the `fetchTodos` action creator dispatches actions conditionally. If we run the app, we can't get it to produce more than three concurrent requests (one for each filter type).

The `isFetching` flag gets reset only when the corresponding `receiveTodos` actions come back, and then we can request the new todos. This is a good way to avoid unnecessary network operations and potential race conditions.

### Updating `fetchTodos`

Since the return value of the thunk is a Promise, we will change our early `return` to be a Promise that resolves immediately. We don't have to do this, but it's convenient for the calling code.

#### Inside `actions/index.js`
```javascript
export const fetchTodos = (filter) => (dispatch, getState) => {
  if (getIsFetching(getState(), filter)) {
    return Promise.resolve();
  }
  // rest of fetchTodos
```

The `thunk` middleware itself does not use this Promise, but it becomes the `return` value of dispatching this action creator, so we can use it inside the component to schedule some code after the asynchronous action has completed.

#### Inside `VisibleTodoList`
```javascript
fetchData() {
  const { filter, fetchTodos } = this.props;
  fetchTodos(filter).then(() => console.log('done!'));
}
```

### Introducing `redux-thunk`

`redux-thunk` is a middleware similar to what we just implemented. To install it, run

`npm install --save redux-thunk`.


With `redux-thunk` installed, we can remove the version of thunk middleware that we just wrote and import `thunk` from `redux-thunk` instead.

[Recap at 2:52 in video](https://egghead.io/lessons/javascript-redux-avoiding-race-conditions-with-thunks)


<p align="center">
<a href="./22-Dispatching_Actions_Asynchronously_with_Thunks.md"><- Prev</a>
<a href="./24-Displaying_Error_Messages.md">Next -></a>
</p>