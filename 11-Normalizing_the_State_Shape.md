# 11. Normalizing the State shape
[Video Link](https://egghead.io/lessons/javascript-redux-normalizing-the-state-shape)

We currently represent the `todos` in the state tree as an array of `todo` objects. However, in the real app we would probably have more than a single array, and `todos` with the same `id`s in different arrays might get out of sync.

#### `todos.js` Before
```javascript
const todos = (state = [], action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [
        ...state,
        todo(undefined, action),
      ];
    case 'TOGGLE_TODO':
      return state.map(t =>
        todo(t, action)
      );
    default:
      return state;
  }
};
```

### Refactoring `todos.js`

We should treat our state as a database, so we are going to keep `todos` in an object indexed by `id`.

We will start by renaming the reducer to `byId`. Now, rather than adding a new item at the end or mapping over every item, we will change the value in the lookup table.

Now both `TOGGLE_TODO` and `ADD_TODO` have the same logic. We want to return a new lookup table where the value of `action.id` is going to be the result of calling the reducer on the previous `action.id` value and the `action`.

```javascript
const byId = (state = {}, action) => {
  switch (action.type) {
    case 'ADD_TODO':
    case 'TOGGLE_TODO':
      return {
        ...state,
        [action.id]: todo(state[action.id], action),
      };
    default:
      return state;
  }
};
```

This is still reducer composition, but with an object instead of an array.

---
**NOTE**:
We are using the Object Spread operator (`...state`). This is not a part of ES6, so we need to install the `transform-object-rest-spread` Babel plugin, and add it to our `.babelrc` file in order for this to work.
---

Anytime the `byId` reducer receives an action, it's going to return a copy of its mapping between the `id`s and the actual todos with an updated `todo` for the current action.

Now we'll add another reducer that keeps track of all the added `id`s.

### Adding an `allIds` Reducer

Now that we keep the todos themselves in the `byId` map, we will have the state of this reducer be an array of `id`s.

This reducer will switch on the action's type, and the only action I care about is `'ADD_TODO'` because if a new todo is added, we want to return a new array of `id`s with the new `id` as the last item.

For any other actions, we just need to return the current state (which is the current array of `id`s).

```javascript
const allIds = (state = [], action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, action.id];
    default:
      return state;
  }
};
```

We still need to export the single reducer from the `todos.js` file, so we use `combineReducers()` again to combine the `byId` and the `allIds` reducers.

```javascript
const todos = combineReducers({
  byId,
  allIds,
});
```

---

_Note_: You can use combined reducers as many times as you like. You don't have to only use it on the top-level reducer. In fact, it's very common that as your app grows, you'll use `combineReducers` in several places.

---

### Updating the `getVisibleTodos` Selector

Now that we have changed the state shape in our reducers, we also need to update the selectors that rely on it.

The `state` object in `getVisibleTodos` is now going to contain `byId` and `allIds` fields, because it corresponds to the `state` of the combined reducer.

Since we don't use an array of todos anymore, we will write a `getAllTodos` selector to create the array for us.

`getAllTodos` will take the current `state` and return all `todos` by mapping `allIds` to the `state`'s `byId` lookup table.

We won't export `getAllTodos` because it will only be used in the current file.

```javascript
const getAllTodos = (state) =>
  state.allIds.map(id => state.byId[id]);
```

We will use this new selector inside our `getVisibleTodo` selector to obtain an array of todos that can be filtered.

`allTodos` is an array of todos just like the components expect, so we can return it from the selector and not worry about changing component code.

```javascript
export const getVisibleTodos = (state, filter) => {
  const allTodos = getAllTodos(state);
  switch (filter) {
    case 'all':
      return allTodos;
    case 'completed':
      return allTodos.filter(t => t.completed);
    case 'active':
      return allTodos.filter(t => !t.completed);
    default:
      throw new Error(`Unknown filter: ${filter}.`);
  }
};
```

### Extracting the `todo` Reducer


The `todos.js` file has grown quite a bit, so it's a good time to extract the todo reducer that manages a single todo into a separate file of its own.

We will create a file called `todo.js` in the same `src/reducers` folder, paste in the implementation. Now we can import it into the `todos.js` file.

#### `todo.js`
```javascript
const todo = (state, action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        id: action.id,
        text: action.text,
        completed: false,
      };
    case 'TOGGLE_TODO':
      if (state.id !== action.id) {
        return state;
      }
      return {
        ...state,
        completed: !state.completed,
      };
    default:
      return state;
  }
};

export default todo;
```

#### `todos.js`
```javascript
import { combineReducers } from 'redux';
import todo from './todo';

const byId = (state = {}, action) => {
  switch (action.type) {
    case 'ADD_TODO':
    case 'TOGGLE_TODO':
      return {
        ...state,
        [action.id]: todo(state[action.id], action),
      };
    default:
      return state;
  }
};

const allIds = (state = [], action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, action.id];
    default:
      return state;
  }
};

const todos = combineReducers({
  byId,
  allIds,
});

export default todos;

const getAllTodos = (state) =>
  state.allIds.map(id => state.byId[id]);

export const getVisibleTodos = (state, filter) => {
  const allTodos = getAllTodos(state);
  switch (filter) {
    case 'all':
      return allTodos;
    case 'completed':
      return allTodos.filter(t => t.completed);
    case 'active':
      return allTodos.filter(t => !t.completed);
    default:
      throw new Error(`Unknown filter: ${filter}.`);
  }
};
```

[Recap at 3:54 in video](https://egghead.io/lessons/javascript-redux-normalizing-the-state-shape)


<p align="center">
<a href="./10-Colocating_Selectors_with_Reducers.md"><- Prev</a>
<a href="./12-Wrapping_dispatch_to_Log_Actions.md">Next -></a>
</p>
