# 24. Displaying Error Messages
[Video Link](https://egghead.io/lessons/javascript-redux-displaying-error-messages)

Sometimes API requests fail, and we will simulate this by `throw`ing inside the fake API client so that it returns a rejected Promise. If we run the app, the loading indicator gets stuck because the `isFetching` flag get set to `true`, but there is no corresponding `receiveTodos` action to set it back to `false` again.

### Fixing It

We'll start by doing some cleanup inside the action creators file (`actions/index.js`).

The `requestTodos` action is never used outside of `fetchTodos`, so we can embed the `requestTodos` object literal inside it. We can do the same thing with `receiveTodos` by copying and pasting it inside of `fetchTodos` where it is dispatched. We will also add the rejection handler as the second argument to our `Promise.then` method.

#### Inside `fetchTodos`
```javascript
return api.fetchTodos(filter).then(
  response => {
    dispatch({
      type: 'RECEIVE_TODOS',
      filter,
      response,
    });
  },
  error => {
    // To be filled in
  }
);
```

### Renaming Actions for Clarity

Since the `fetchTodos` action creator dispatches several actions, we will rename them to be more consistent:
  * `'REQUEST_TODOS'` becomes `'FETCH_TODOS_REQUEST'` for requesting the todos
  * `'RECEIVE_TODOS'` becomes `'FETCH_TODOS_SUCCESS'` for successfully fetching the todos
  * Add `'FETCH_TODOS_FAILURE'` in our `error` handler when failing to fetch the todos.

Our `error` handler will also be passed two additional pieces of data: the `filter` and the `message` that can be read with `error.message` if specified. We will use `'Something went wrong.'` as a fallback.

#### Updated `fetchTodos` `return`
```javascript
return api.fetchTodos(filter).then(
  response => {
    dispatch({
      type: 'FETCH_TODOS_SUCCESS',
      filter,
      response,
    });
  },
  error => {
    dispatch({
      type: 'FETCH_TODOS_FAILURE',
      filter,
      message: error.message || 'Something went wrong.',
    });
  }
);
```

Now our `fetchTodos` action creator handles all the cases, and we can remove the old action creators that are now inlined (`requestTodos` and `receiveTodos`).

### Updating our Reducers

Since we changed action types, we now need to change the corresponding reducers.

Our `ids` reducer needs to handle `FETCH_TODOS_SUCCESS` instead of `RECEIVE_TODOS`.

The `isFetching` reducer needs to handle `FETCH_TODOS_REQUEST` instead of `REQUEST_TODOS`, and `FETCH_TODOS_SUCCESS` instead of `RECEIVE_TODOS`.

We will also handle `FETCH_TODOS_FAILURE` by returning `false` so our loading indicator won't get stuck.

The last reducer we need to change is `byId`, where we replace `RECEIVE_TODOS` with `FETCH_TODOS_SUCCESS`.

#### Updated `isFetching` Reducer
```javascript
const isFetching = (state = false, action) => {
    if (filter !== action.filter) {
      return state;
    }
    switch (action.type) {
      case 'FETCH_TODOS_REQUEST':
        return true;
      case 'FETCH_TODOS_SUCCESS':
      case 'FETCH_TODOS_FAILURE':
        return false;
      default:
        return state;
    }
  };
```

With these changes, the loading indicator won't get stuck because a corresponding failure action fires, resetting `isFetching` back to `false`.

### Displaying the Error

We'll create a new file `FetchError.js` in our `components` directory.

After `import`ing `React`, we'll create a new functional stateless component `FetchError` that will take two props: a `message` string, and an `onRetry` function. This component will be the default export for this file.

The rendered `<div>` will contain an error saying that something bad happened (including the message that is passed in the props), and a button that when clicked will invoke the `onRetry` callback prop so that the user can retry fetching the data.

##### `FetchError` Component
```javascript
const FetchError = ({ message, onRetry }) => (
  <div>
    <p>Could not fetch todos. {message}</p>
    <button onClick={onRetry}>Retry</button>
  </div>
);
```

### Adding `FetchError` to `VisibleTodoList`

We need to `import FetchError` inside of `VisibleTodoList`, then update the `render` method.

In order to get the error message, we need to destructure it from the `props` of the `VisibleTodoList` component.

```javascript
// Inside VisibleTodoList
render() {
  const { isFetching, errorMessage, toggleTodo, todos } = this.props;
  ...
```

We will also add another condition inside of `render` saying that "if we have an error message in our props and we have no todos to display, then return the `FetchError` component.

The `FetchError` component itself wants a `message` prop, which will be passed the `errorMessage` prop I just de-structured. We will also provide an `onRetry` callback prop that we will pass an error function that calls `this.fetchData` to restart the data fetching process.

```javascript
// Inside VisibleTodoList's `render()`
if (errorMessage && !todos.length) {
     return (
       <FetchError
         message={errorMessage}
         onRetry={() => this.fetchData()}
       />
     );
   }
```

We need to add `errorMessage` into `VisibleTodoList`'s `mapStateToProps` in order to make it available. Following the same pattern used with `isFetching`, we will get the `errorMessage` prop by calling a selector called `getErrorMessage` and pass in the `state` of the app and the `filter`.

```javascript
// Inside VisibleTodoList
const mapStateToProps = (state, { params }) => {
  const filter = params.filter || 'all';
  return {
    isFetching: getIsFetching(state, filter),
    errorMessage: getErrorMessage(state, filter),
    todos: getVisibleTodos(state, filter),
    filter,
  };
};
```

### Implementing `getErrorMessage`
We need to add `getErrorMessage` to `VisibleTodoList`'s reducer import at the top of the file:
`import { getVisibleTodos, getErrorMessage, getIsFetching } from '../reducers';`

Now inside of our root reducers file (/reducers/index.js) we create our `getErrorMessage` selector by copying, pasting, and refactoring our `getIsFetching` selector.

#### Creating the Selector
```javascript
export const getErrorMessage = (state, filter) =>
  fromList.getErrorMessage(state.listByFilter[filter]);
```

#### Updating `createList`

Inside `createList.js`, we'll add a new exported selector called `getErrorMessage` that takes the state of the list and returns a property called error message.
```javascript
export const getErrorMessage = (state) => state.errorMessage;
```

We'll now declare a new reducer called `errorMessage` with the initial `state` of `null`.  We do this because a reducer cannot have an undefined initial state, so we have to make its absence explicit.

Like in the other reducers in this file, we want to skip any actions with the filter that don't match the `filter` specified as an argument to `createList`. When the filter _does_ match, we want to handle a few actions:

  * Display an error message if there's a failure
  * Clear the error message if the user retries their request
  * Return the current state for any other action

The `errorMessage` reducer needs to be added to `combineReducers` as well.

##### Completed `errorMessage` Reducer
```javascript
const errorMessage = (state = null, action) => {
    if (filter !== action.filter) {
      return state;
    }
    switch (action.type) {
      case 'FETCH_TODOS_FAILURE':
        return action.message;
      case 'FETCH_TODOS_REQUEST':
      case 'FETCH_TODOS_SUCCESS':
        return null;
      default:
        return state;
    }
  };

  return combineReducers({
    ids,
    isFetching,
    errorMessage,
  });
```

### Updating the API
Instead of having our API throw the error every time, we'll have it throw randomly so we can try out our "retry" button.

[Demonstration and recap at 6:43 in video](https://egghead.io/lessons/javascript-redux-displaying-error-messages)


<p align="center">
<a href="./23-Avoiding_Race_Conditions_with_Thunks.md"><- Prev</a>
<a href="./25-Creating_Data_on_the_Server.md">Next -></a>
</p>