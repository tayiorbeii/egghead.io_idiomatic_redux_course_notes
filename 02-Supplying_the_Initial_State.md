# 02\. Supplying the Initial State

[Video Link](https://egghead.io/lessons/javascript-redux-supplying-the-initial-state)

When you create a Redux store, its initial state is determined by the root reducer.

In this case, the root reducer is the result of calling `combineReducers` on `todos` and `visibilityFilter`.

##### _Inside `src/index.js`_
```javascript
const store = createStore(todoApp)
```


##### _Inside `src/reducers/index.js`_
```javascript
const todoApp = combineReducers({
  todos,
  visibilityFilter
})
```

Since reducers are autonomous, each of them specifies its own initial state as the default value of the `state` argument.

In `const todos = (state = [], action) ...` the default state is an empty array.

The default state value in `const visibilityFilter = (state = 'SHOW_ALL', action) ...` is a string.

Therefore, our initial state of the combined reducer is going to be an object containing an empty array under the `todos` key, and a string `'SHOW_ALL'` under the `visibilityFilter` key:


##### Initial State:
```javascript
const todoApp = combineReducers({
    todos,
    visibilityFilter
})
```

However, sometimes we want to load data into the store synchronously before the application starts.

For example, there may be Todos left from the previous session.

Redux lets us pass a `persistedState` as the second argument to the `createStore` function:
```javascript
const persistedState = {
  todos: [{
    id: 0,
    text: 'Welcome Back!',
    completed: false
  }]
}

const store = createStore(
  todoApp,
  persistedState
)
```

When passing in a `persistedState`, it will overwrite the default values set in the reducer as applicable. In this example, we've provided `todos` with an array, but since we didn't specify a value for `visibilityFilter`, the default `'SHOW_ALL'` will be used.

Since the `persistedState` that we provided as a second argument to `createStore()` was obtained from Redux itself, it doesn't break encapsulation of reducers.

It may be tempting to supply an initial state for your entire store inside of `persistedState`, but it is not recommended. If you were to do this, it would become more difficult to test and change your reducers later.

#### [Recap at 1:42 in video](https://egghead.io/lessons/javascript-redux-supplying-the-initial-state)


<p align="center">
  <a href="./01-Simplifying_the_Arrow_Functions.md"><- Prev</a>
  <a href="./03-Persisting_the_State_to_the_Local_Storage.md">Next -></a>
</p>
