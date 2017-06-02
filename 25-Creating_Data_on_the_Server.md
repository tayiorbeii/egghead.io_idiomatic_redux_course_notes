# 25. Creating Data on the Server
[Video Link](https://egghead.io/lessons/javascript-redux-creating-data-on-the-server)

### Updating the Fake API

Some new functions were added to our fake API client for the next few lessons.

The first new fake API endpoint is called `addTodo`. It simulates a network connection, and then it creates a new `todo` object. It puts the todo with the given text into the fake database, and it returns the `todo` object, just like REST endpoints normally do.

#### `addTodo` Endpoint
```javascript
export const addTodo = (text) =>
  delay(500).then(() => {
    const todo = {
      id: v4(),
      text,
      completed: false,
    };
    fakeDatabase.todos.push(todo);
    return todo;
  });
```

The second fake API endpoint I is called `toggleTodo`. It also simulates a network connection, and it finds the corresponding todo in the fake database and flips its `completed` field, also returning the `todo` as the response.

#### `toggleTodo` Endpoint
```javascript
export const toggleTodo = (id) =>
  delay(500).then(() => {
    const todo = fakeDatabase.todos.find(t => t.id === id);
    todo.completed = !todo.completed;
    return todo;
  });
```

### Updating the "Add Todo" Process

In this lesson, we will make the add todo button called the `addTodo` fake API endpoint.

Inside our action creators file, we will no longer need the `v4` function from `node-uuid` because the id generation will now occur on the server.

The `addTodo` action creator will be changed to be a thunk action creator, so we will add `dispatch` as a curried argument. The thunk will call the API's `addTodo` endpoint with the given text, and wait for the response to come back.

When the response is available, it will dispatch an action with the type `'ADD_TODO_SUCCESS'`, and the server response.

##### Updated `addTodo` Action
```javascript
export const addTodo = (text) => (dispatch) =>
  api.addTodo(text).then(response => {
    dispatch({
      type: 'ADD_TODO_SUCCESS',
      response,
    });
  });
```

#### Updating the `byId` Reducer

Since the newly added todo will be part of the server response, we need to change the `byId` reducer to merge the todo into the lookup table that it manages.

I am adding a new case for the `'ADD_TODO_SUCCESS'` action. We'll use the object spread operator to create a new version of the lookup table, where under the action response ID key, there is a new todo object that I read from action response.

```javascript
// Inside reducers/byId.js
const byId = (state = {}, action) => {
  switch (action.type) {
    case 'FETCH_TODOS_SUCCESS':
      const nextState = { ...state };
      action.response.forEach(todo => {
        nextState[todo.id] = todo;
      });
      return nextState;
    case 'ADD_TODO_SUCCESS': // Our new case
      return {
        ...state,
        [action.response.id]: action.response,
      };
    default:
      return state;
  }
};
```

However, we have not updated the list by filter, so all still only has three IDs in the list. If I go to another tab, the new todo appears because its ID is now included in the list of fetched IDs, and similarly, if I go back to the previous tab, it appears now because the data has been re-fetched.

#### Updating `createList`

Since the list of IDs for each tab is managed by a reducer defined inside `createlist.js`, we need to update our `ids` reducer to handle the `'ADD_TODO_SUCCESS'` action.

When we receive a confirmation that the todo has been added on the server, we can return a new list of IDs with existing IDs in the beginning, and a newly added ID at the end.

Unlike the other actions, `'ADD_TODO_SUCCESS'` does not have a `filter` property on the `action` object, our current `if (filter !== action.filter)` check inside of `ids` would fail. Because of this, we will replace the existing check with different checks in different cases.

For `FETCH_TODOS_SUCCESS`, we want to replace the fetched IDs if the filter in the action matches the filter of this list.

However, for `ADD_TODO_SUCCESS`, we want the newly added todo to appear in every list except the completed filter, because a newly added todo is not completed.

```javascript
const createList = (filter) => {
  const ids = (state = [], action) => {
    switch (action.type) {
      case 'FETCH_TODOS_SUCCESS':
        return filter === action.filter ?
          action.response.map(todo => todo.id) :
          state;
      case 'ADD_TODO_SUCCESS':
        return filter !== 'completed' ?
          [...state, action.response.id] :
          state;
      default:
        return state;
    }
  };
```

[Demonstration and recap at 3:36 in video](https://egghead.io/lessons/javascript-redux-creating-data-on-the-server)


<p align="center">
<a href="./24-Displaying_Error_Messages.md"><- Prev</a>
<a href="./26-Normalizing_API_Responses_with_normalizr.md">Next -></a>
</p>