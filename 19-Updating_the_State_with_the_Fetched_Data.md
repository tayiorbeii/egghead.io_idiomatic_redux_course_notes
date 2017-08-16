# 19. Updating the State with the Fetched Data
[Video Link](https://egghead.io/lessons/javascript-redux-updating-the-state-with-the-fetched-data#/tab-transcript)


In the current implementation of `getVisibleTodos` inside of `todos.js`, we keep all todos in memory. We have an array of all `id`s ever and we get the array of todos that we can filter according to the filter passed from React Router.

#### Current `getVisibleTodos`
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

However, this only works correctly if all the data from the server is already available in the client, which is usually not the case with applications that fetch something. If we have thousands of todos on the server, it would be impractical to fetch them all and filter them on the client.

### Refactoring `getVisibleTodos`

Rather than keep one big list of `id`s, we'll keep a list of `id`s for every filter's tab so that they can be stored separately and filled according to the actions with the fetched data.

We'll remove the `getAllTodos` selector because we won't have access to `allTodos`. We also don't need to filter on the client anymore because we will use the list of todos provided by the server. This means we can remove our `switch` statement from the current implementation.

Instead of reading from `state.allIds`, we will read the IDs from `state.IdsByFilter[filter]`. Then we will map the `id`s to the `state.ById` lookup table to get the actual todos.

#### Updated `getVisibleTodos`
```javascript
export const getVisibleTodos = (state, filter) => {
  const ids = state.idsByFilter[filter];
  return ids.map(id => state.byId[id]);
};
```

### Refactoring `todos`

The selector now expects `idsByFilter` and `byId` to be part of the combined `state` of the `todos` reducer.

#### `todos` Reducer Before:
```javascript
const todos = combineReducers({
  byId,
  allIds
});
```

The `todos` reducer used to combine the lookup table and a list of `allIds`. Now, though, we'll replace the lis of `allIds` with the list of `idsByFilter`, which will be a new combined reducer.

#### `todos` Reducer After:
```javascript
const todos = combineReducers({
  byId,
  idsByFilter
});
```

### Creating `idsByFilter`

`idsByFilter` combines a separate list of `id`s for every filter. So it's `allIds` for the `all` filter, `activeIds` for the `active` filter, and `completedIDs` for the `completed` filter.

```javascript
const idsByFilter = combineReducers({
  all: allIds,
  active: activeIds,
  completed: completedIds,
});
```

### Updating the `allIds` Reducer

The original `allIDs` reducer managed an array of IDs and the `ADD_TODO` action.

We are going to take this responsibility away for now, because for now we want to teach it to respond to the data fetched from the server.

#### `allIds` Before:
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

We'll start by renaming `ADD_TODO` to `RECEIVE_TODOS`. In order to handle the `RECEIVE_TODOS` action, we want to return a new array of todos that we'll get from the server response. We'll map this new array of todos to a function that just selects an `id` from the `todo`. Recall that we decided to keep all IDs separate from active IDs and completed IDs, so they are fetched completely independently.

#### `allIds` After:
```javascript
const allIds = (state = [], action) => {
  switch (action.type) {
    case 'RECEIVE_TODOS':
      return action.response.map(todo => todo.id);
    default:
      return state;
  }
};
```

#### Creating the `activeIds` Reducer

Our `activeIds` reducer will also keep track of an array of `id`s, but only for `todos` on the active tab. We will need to handle the `RECEIVE_TODOS` action in exactly the same way as the `allIds` reducer before it.

```javascript
const activeIds = (state = [], action) => {
  switch (action.type) {
    case 'RECEIVE_TODOS':
      return action.response.map(todo => todo.id);
    default:
      return state;
  }
};
```

### Updating the Correct Array

Both `activeIds` and `allIds` need to return a new `state` when the `RECEIVE_TODOS` action fires, but we need to have a way of telling which `id` array we should update.

If you recall the `RECEIVE_TODOS` action, you might remember that we passed the `filter` as part of the action. This lets us compare the `filter` inside the action with a `filter` corresponding to the reducer.

The `allIds` reducer is only interested in the actions with the `all` filter, and the `activeIds` is only interested in the `active` filter.

#### `activeIds` Reducer
```javascript
const activeIds = (state = [], action) => {
  if (action.filter !== 'active') {
    return state;
  }
  // rest of code as above
```
_Repeat for `allIds` but remember to replace `active` with `all`_


### Creating the `completedIds` Reducer
This reducer is the same as our other filter reducers, but for the `complete` filter.

```javascript
const completedIds = (state = [], action) => {
  if (action.filter !== 'completed') {
    return state;
  }
  switch (action.type) {
    case 'RECEIVE_TODOS':
      return action.response.map(todo => todo.id);
    default:
      return state;
  }
};
```

### Updating the `byId` Reducer

Now that we have reducers that managing the `id`s, we need to update the `byId` reducer to actually handle the new `todos` from the response.

#### `byId` Before:
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

We can start by removing the existing `case`s because the data does not live locally anymore. Instead, we will handle the `RECEIVE_TODOS` action just in the other reducers.

Then we'll create `nextState`, a shallow copy of the `state` object which corresponds to the lookup table. We want to iterate through every `todo` object in the `response` and put it into our `nextState`.

We'll replace whatever is in `nextState`'s entry for `todo.id` with the new `todo` we just fetched.

Finally, we'll return the next version of the lookup table from our reducer.

#### `byId` After:
```javascript
const byId = (state = {}, action) => {
  switch (action.type) {
    case 'RECEIVE_TODOS': // eslint-disable-line no-case-declarations
      const nextState = { ...state };
      action.response.forEach(todo => {
        nextState[todo.id] = todo;
      });
      return nextState;
    default:
      return state;
  }
};
```

**Note:** Normally the assignment operation is a mutation. However, in this case it's fine because `nextState` is a shallow copy, and we're only assigning one level deep. Our function stays pure because we're not modifying any of the original state objects.

### Finishing Up

As the last step, we can remove the import of `todo.js` as well as the file itself from our project, because the logic for adding and toggling todos will be implemented as API calls to the server in the future lessons.

[Recap at 5:22 in video](https://egghead.io/lessons/javascript-redux-updating-the-state-with-the-fetched-data)


<p align="center">
<a href="./18-Applying_Redux_Middleware.md"><- Prev</a>
<a href="./20-Refactoring_the_Reducers.md">Next -></a>
</p>