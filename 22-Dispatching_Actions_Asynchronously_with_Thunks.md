# 22. Dispatching Actions Asynchronously with Thunks
[Video Link](https://egghead.io/lessons/javascript-redux-dispatching-actions-asynchronously-with-thunks)

To show the loading indicator in our current implementation, we dispatch the `requestTodos` action before we fetch the todos with `fetchTodos`. It would be great if we could make `requestTodos` dispatch automatically when we fetch the todos because we never want them to fire separately.

#### `VisibleTodoList.js` Before
```javascript
// inside of `VisibleTodoList`
fetchData() {
  const { filter, fetchTodos, requestTodos } = this.props;
  requestTodos(filter);
  fetchTodos(filter);
}
```


To start, we will remove the explicit `requestTodos` dispatch from the component. Now we no longer need to `export` the `requestTodos` action creator inside of `actions/index.js` (but don't erase the code for it).

Our goal is to dispatch `requestTodos()` when we start fetching, and `receiveTodos()` when they finish fetching, but our `fetchTodos` action creator only resolves through the `receiveTodos` action.

#### Current `fetchTodos` Action Creator
```javascript
export const fetchTodos = (filter) =>
  api.fetchTodos(filter).then(response =>
    receiveTodos(filter, response)
  );
```

An action Promise resolves through to a single action at the end, but we want an abstraction that represents multiple actions dispatched over a period of time. This is why rather than returning a Promise, I want to return a function that accepts a dispatch callback argument.

This change lets us call `dispatch()` as many times as we want at any point of time during the async operation. We can dispatch the `requestTodos` action in the beginning, then when the Promise resolves we can explicitly dispatch another `receiveTodos` action.

#### Updated `fetchTodos`
```javascript
export const fetchTodos = (filter) => (dispatch) => {
  dispatch(requestTodos(filter));

  return api.fetchTodos(filter).then(response => {
    dispatch(receiveTodos(filter, response));
  });
};
```

This is more typing than returning a Promise, but it gives us more flexibility. Since a Promise can only express one async value, `fetchTodos` now returns a function with a callback argument so that it can call it multiple times during the async operation.

### Introducing Thunks

Functions returned from other functions like in `fetchTodos` are often called thunks. Now we're going to implement a middleware to support using thunks in our code.

### Updating `configureStore.js`

Inside of `configureStore.js` we'll remove the `redux-promise` middleware and replace it with the `thunk` middleware we'll write now.

The `thunk` middleware supports the dispatching of thunks. It takes the `store`, the next middleware (`next`), and the `action` as arguments, just like any other middleware.

If `action` is a function instead of an action, we're going to assume that this is a thunk that wants the `dispatch` function to be injected into it. In this case, we'll call the action with `store.dispatch`.

Otherwise, we just return the result of passing the action to the next middleware in chain.

#### `thunk` Middleware in `configureStore.js`
```javascript
const thunk = (store) => (next) => (action) =>
  typeof action === 'function' ?
    action(store.dispatch) :
    next(action);
```

As a final step, we need to add our new `thunk` middleware to the array of middlewares inside `configureStore` so that it gets applied to the store.

```javascript
const configureStore = () => {
  const middlewares = [thunk];
  // rest of configureStore
```

[Recap at 2:45 in video](https://egghead.io/lessons/javascript-redux-dispatching-actions-asynchronously-with-thunks)


<p align="center">
<a href="./21-Displaying_Loading_Indicators.md"><- Prev</a>
<a href="./23-Avoiding_Race_Conditions_with_Thunks.md">Next -></a>
</p>