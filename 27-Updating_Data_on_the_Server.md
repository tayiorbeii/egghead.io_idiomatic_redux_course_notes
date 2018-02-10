# 27. Updating Data on the Server
[Video Link](https://egghead.io/lessons/javascript-redux-updating-data-on-the-server)

We'll start by changing the `toggleTodo` action creator to be a thunk action creator, so we add `dispatch` as a curried argument. Next, we'll call the `toggleTodo` API endpoint, and wait for the response to come back.

When the response is available, we will dispatch an action with the type `'TOGGLE_TODO_SUCCESS'`, and the response. We'll use `normalize` again by passing the original `response` as the first argument, and the todo schema as the second argument.

##### Updated `toggleTodo` Action Creator
```javascript
export const toggleTodo = (id) => (dispatch) =>
  api.toggleTodo(id).then(response => {
    dispatch({
      type: 'TOGGLE_TODO_SUCCESS',
      response: normalize(response, schema.todo),
    });
  });
```

### Updating the `ids` Reducer

Inside of `createList.js` we will add a new case for the `'TOGGLE_TODO_SUCCESS'` action.

We'll extract the code for this case into a separate function called `handleToggle`, and pass in the `state` and the `action`.

```javascript
// Inside the `ids` reducer
  case 'TOGGLE_TODO_SUCCESS':
    return handleToggle(state, action);
```

We'll put our `handleToggle` function above the `ids` reducer. It accepts the `state` (an array of ids) and the `'TOGGLE_TODO_SUCCESS'` action.

We will destructure the `result` as the `toggledId` and the `entities` from the normalized response. Next, we will read the `completed` value from the `todo`, which I get by referencing `entities.todos` by the `toggledId`.

There are two cases in which we want to remove the todo from the list:
 * The `completed` field is `true` but the `filter` is `active`
 * `completed` is `false` but the `filter` is `completed`

When `shouldRemove` is `true`, we want to return a copy of the list that does not contain the id of the todo that was just toggled. We accomplish this by filtering the list by id and only leave the ones that have a different id. Otherwise, we return the original array.

##### Completed `handleToggle` Function
```javascript
const handleToggle = (state, action) => {
  const { result: toggledId, entities } = action.response;
  const { completed } = entities.todos[toggledId];
  const shouldRemove = (
    (completed && filter === 'active') ||
    (!completed && filter === 'completed')
  );
  return shouldRemove ?
    state.filter(id => id !== toggledId) :
    state;
};
```

[Demonstration and recap at 3:20 in video](https://egghead.io/lessons/javascript-redux-updating-data-on-the-server)


<p align="center">
  <a href="./26-Normalizing_API_Responses_with_normalizr.md"><- Prev</a>
</p>