# 26. Normalizing API Responses with `normalizer`
[Video Link](https://egghead.io/lessons/javascript-redux-normalizing-api-responses-with-normalizr)

The `byId` reducer currently has to handle server actions differently, because they have different response shapes.

For example, the `'FETCH_TODOS_SUCCESS'` action's response is an array of todos. This array has be to iterated over and merged one at a time into the next state.

The response for `'ADD_TODO_SUCCESS'` is the single todo that has just been added, and this single todo has to be merged in a different way.

Instead of adding new cases for every new API call, I want to normalize the responses so the response shape is always the same.

### Installing `normalizr`

`normalizr` is a utility library that will help us normalize API responses to have the same shape.

`$ npm install --save normalizr`

###  Creating `schema.js`
We'll create a new file `schema.js` inside of our `actions` directory.

We'll start by importing a `Schema` constructor and a function called `arrayOf` from `normalizr`.

Our first exported Schema will be for the `todo` objects, and we'll specify `todos` as the name of the dictionary in the normalized response.

Our next schema called `arrayOfTodos` corresponds to the responses that contain arrays of `todo` objects.

```javascript
import { schema } from 'normalizr'

export const todo = new schema.Entity('todos');
export const arrayOfTodos = new schema.Array(todo);
```

### Updating our Action Creators

Inside of `actions/index.js`, we'll add a named import for a function called `normalize` that we import from `normalizr`. We also add a namespace import for all the Schemas we defined in the schema file.

Inside of the `FETCH_TODOS_SUCCESS` callback, we'll add a "normalized response" log so that I can see what the normalized response looks like. We call the `normalize` function with the original `response` as the first argument, and the corresponding schema (in this case, `arrayOfTodos`) as the second argument.

```javascript
return api.fetchTodos(filter).then(
  response => {
    console.log(
      'normalized response',
      normalize(response, schema.arrayOfTodos)
    )
    dispatch({
      type: 'FETCH_TODOS_SUCCESS',
      filter,
      response,
    });
  },
```

We'll update `addTodo` in a similar manner:
```javascript
export const addTodo = (text) => (dispatch) =>
  api.addTodo(text).then(response => {
    console.log(
      'normalized response',
      normalize(response, schema.todo)
    )
    dispatch({
      type: 'ADD_TODO_SUCCESS',
      responsed,
    });
  });
```

### Comparing Responses

At this point, the response in the action is an array of to-do objects, but our normalized response for `'FETCH_TODOS_SUCCESS'` is an object that contains two fields: `entities` and `result`.

`entities` contains a normalized dictionary called `todos` that contains every `todo` in the response by its id. `normalizr` found these `todo` objects in the response by following the `arrayOfTodos` schema. Conveniently, they are indexed by IDs, so they will be easy to merge into the lookup table.

The second field is the `result`, which is an array of `todo` IDs. They are in the same order as the `todos` in the original response array. However, `normalizr` replaced each `todo` with its ID, and moved every todo into the `todos` dictionary.


### Finishing our Action Creator Updates

We will now change the action creators so that they pass the normalized response in the response field, instead of the original response.

##### Before:
```javascript
return api.fetchTodos(filter).then(
  response => {
    console.log(
      'normalized response',
      normalize(response, schema.arrayOfTodos)
    )
    dispatch({
      type: 'FETCH_TODOS_SUCCESS',
      filter,
      response,
    });
  },
```

##### After:
```javascript
return api.fetchTodos(filter).then(
    dispatch({
      type: 'FETCH_TODOS_SUCCESS',
      filter,
      response: normalize(response, schema.arrayOfTodos),
    });
  },
```

### Updating the Reducers

We can delete the special cases in our `byId` reducer, because the response shape has been normalized. Instead of switching by action type, we will check to see if the action has a response object on it.

##### `byId` Reducer Before:
```javascript
const byId = (state = {}, action) => {
  switch (action.type) {
    case 'FETCH_TODOS_SUCCESS': // eslint-disable-line no-case-declarations
      const nextState = { ...state };
      action.response.forEach(todo => {
        nextState[todo.id] = todo;
      });
      return nextState;
    case 'ADD_TODO_SUCCESS':
      return {
        ...state,
        [action.response.id]: action.response,
      };
    default:
      return state;
  }
};
```

We will return a new version of the lookup table that contains all existing entries, as well as any entries inside `entities.todos` in the normalized response. For other actions, I will return the lookup table as it is.

##### `byId` Reducer After:
```javascript
const byId = (state = {}, action) => {
  if (action.response) {
    return {
      ...state,
      ...action.response.entities.todos,
    };
  }
  return state;
};
```

Now we need to update the `ids` reducer inside of `createList.js` for our new `action.response` shape.

##### `ids` Reducer Before:
```javascript
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

Now, the action response has a `result` field, which is either an array of `id`s (in the case of `'FETCH_TODOS_SUCCESS'`), or a single `id` of the fetched todo (in the case of `'ADD_TODO_SUCCESS'`).

##### `ids` Reducer After
```javascript
const ids = (state = [], action) => {
   switch (action.type) {
     case 'FETCH_TODOS_SUCCESS':
       return filter === action.filter ?
         action.response.result :
         state;
     case 'ADD_TODO_SUCCESS':
       return filter !== 'completed' ?
         [...state, action.response.result] :
         state;
     default:
       return state;
   }
 };
```

[Demonstration and recap at 5:33 in video](https://egghead.io/lessons/javascript-redux-normalizing-api-responses-with-normalizr)


<p align="center">
<a href="./25-Creating_Data_on_the_Server.md"><- Prev</a>
<a href="./27-Updating_Data_on_the_Server.md">Next -></a>
</p>