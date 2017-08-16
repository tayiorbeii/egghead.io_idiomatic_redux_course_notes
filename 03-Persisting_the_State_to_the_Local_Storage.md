# 03. Persisting the State to the Local Storage
[Video Link](https://egghead.io/lessons/javascript-redux-persisting-the-state-to-the-local-storage)

In this example, we want to be able to persist the state of the application using the browser's `localStorage` API.

We will write a function called `loadState()` and a new model called `localStorage.js`.

The `loadState` function is going to look into `localStorage` by key, retrieve a string, and try to parse it as JSON. This code needs to be wrapped into a `try/catch` because it's possible that the user's browser disallows usage of the `localStorage` API.

####`localStorage.js`
```javascript
export const loadState = () => {
  try {
    const serializedState = localStorage.getItem('state');
    if (serializedState === null) {
      return undefined;
    }
    return JSON.parse(serializedState);
  } catch (err) {
    return undefined;
  }
};
```

In our `loadState` function, if the `serializedState` is `null`, it means that the key doesn't exist, so we return `undefined` which lets the reducers set the state instead.

However, if the `serializedState` string exists, we'll use `JSON.parse(serializedState)` in order to return it into the `state` object.

Since we have a function for loading state, we should have one for saving state to localStorage:

```javascript
export const saveState = (state) => {
  try {
    const serializedState = JSON.stringify(state);
    localStorage.setItem('state', serializedState);
  } catch (err) {
    // Ignore write errors.
  }
};
```

Our `saveState` function accepts our `state` as an argument, and does the exact opposite thing as our `loadState` function.

First, we use `JSON.stringify(state)` to serialize our state to a string. This will only work if the state is serializable, but if you're following suggested Redux practices, you'll be good to go.

We catch any errors, since either `JSON.stringify()` or `localStorage.setItem()` can fail.

### Using `localStorage.js`
Back in our `index.js` file, we will import the functions we just wrote:
```javascript
import { loadState, saveState } from './localStorage'
```

In order to save our state any time the store changes, we will use the `store`'s `subscribe()` method to add a listener that will be invoked on any state change, passing in the current state of the store into the `saveState` function:

```javascript
// Inside of index.js ...
store.subscribe(() => {
  saveState(store.getState())
})
```

Now we have our state being preserved across reloads. However, it isn't just our complete & incomplete Todo items being tracked, but the visibility filter as well. This isn't ideal, since in most cases we would want to persist just the data and not the UI as well.

To fix this, we'll adjust `store.subscribe()`:
```javascript
store.subscribe(() => {
  saveState({
    todos: store.getState().todos
  })
})
```
Now that we are only saving the `todos` portion of our state, when we refresh the page our `visibilityFilter` will be set to the default of `'SHOW_ALL'` by its reducer.

### But we have a bug...
With the way our code is currently written, when we try to add a new Todo item, React will error with `Encountered two children with the same key, 0`.

This is because in our `TodoList` component we use `todo.id` as our key, which is assigned in the `addTodo` action creator inside of `actions.js`. The `addTodo` action creator uses a local variable `nextTodoId` that is assigned to `0` by default.

##### `addTodo` Before:
```javascript
let nextTodoId = 0

export const addTodo = (text) => ({
  type: 'ADD_TODO',
  id: (nextTodoId++).toString(),
  text
})
```

To get around this, we will use an npm module called `node-uuid`:

`$ npm install --save node-uuid`

To use the module, we import `v4` from `node-uuid` and call it instead of `(nextTodoId++)`. _Note: `v4` is just the name of the standard._

##### `addTodo` After:
```javascript
import { v4 } from 'node-uuid'

export const addTodo = (text) => ({
  type: 'ADD_TODO',
  id: v4(),
  text
})
```
Now all of our Todo items are persisted throughout reloads.

### Throttling `saveState()`
We currently call `saveState()` inside the subscribe listener so it is called every time the storage state changes. We want to avoid calling it too often because it uses the expensive `stringify` operation.

We will fix this by using another npm module `lodash` that includes a `throttle` utility.

`$ npm install --save lodash`


#### `store.subscribe()` Before:
```javascript
store.subscribe(() => {
  saveState({
    todos: store.getState().todos
  })
})
```

By wrapping our callback in a `throttle` call, we can insure that the inner function we pass is not going to be called more often than our specified number of milliseconds.

We'll import just `throttle` directly from `lodash`, instead of bringing in the entire library to use a single function.

#### `store.subscribe()` After:
```javascript
// top of index.js
import throttle from 'lodash/throttle'
.
.
.
store.subscribe(throttle(() => {
  saveState({
    todos: store.getState().todos
  })
}, 1000))
```

Now, even if the store gets updated really fast, we have a guarantee that we only write to `localStorage` once a second at most.

#### [Recap at 6:05 in video](https://egghead.io/lessons/javascript-redux-persisting-the-state-to-the-local-storage#/tab-transcript)


<p align="center">
<a href="./02-Supplying_the_Initial_State.md"><- Prev</a>
<a href="./04-Refactoring_the_Entry_Point.md">Next -></a>
</p>