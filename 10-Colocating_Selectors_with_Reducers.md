# 10. Colocating Selectors with reducers
[Video Link](https://egghead.io/lessons/javascript-redux-colocating-selectors-with-reducers?series=building-react-applications-with-idiomatic-redux)

Inside of `VisibleTodoList`, the `mapStateToProps` function uses the `getVisibleTodos` function, and it passes the slice of the `state` corresponding to the `todos`. However, if we ever change the state's structure, we have to remember to update this whole side.

In order to clean this up, we can move the `getVisibleTodos` function out of the view layer and place it inside of the file that contains our `todos` reducer. We do this because the `todos` reducer knows the most about the internal structure of the state's `todos`.

#### `VisibleTodoList` Before
```javascript
const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case 'all':
      return todos;
    case 'completed':
      return todos.filter(t => t.completed);
    case 'active':
      return todos.filter(t => !t.completed);
    default:
      throw new Error(`Unknown filter: ${filter}.`);
  }
};

const mapStateToProps = (state, { params }) => ({
  todos: getVisibleTodos(state.todos, params.filter || 'all'),
});
```

_Note: the explanations in the video require a version of `react-router` previous to the 4.0.0. Starting in that version some changes have been included which require this slightly different syntax:_

#### `VisibleTodoList` Before (react-router v4.0.0 or superior)
```javascript
const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case 'all':
      return todos;
    case 'completed':
      return todos.filter(t => t.completed);
    case 'active':
      return todos.filter(t => !t.completed);
    default:
      throw new Error(`Unknown filter: ${filter}.`);
  }
};

const mapStateToProps = (state, { match }) => ({
  todos: getVisibleTodos(state.todos, match.params.filter || 'all'),
});
```

### Updating our Reducer

We are going to move our `getVisibleTodos` implementation into the file with the reducers, and make it a named export.

The convention we follow is simple. The default export is always the reducer function, but any named export starting with `'get'` is a function that prepares the data to be displayed by the UI. We usually call these functions _selectors_ because they select something from the current state.

In the reducers, the `state` argument corresponds to the state of this particular reducer, so we will follow the same convention for selectors. The `state` argument corresponds to the state of the exported reducer in this file.

#### Inside `src/reducers/todos.js`
```javascript
export const getVisibleTodos = (state, filter) => {
  switch (filter) {
    case 'all':
      return state;
    case 'completed':
      return state.filter(t => t.completed);
    case 'active':
      return state.filter(t => !t.completed);
    default:
      throw new Error(`Unknown filter: ${filter}.`);
  }
};
```

### Updating the Root Reducer
Inside of `VisibleTodoList` we still depend on the state structure because we read the `todos` from the state, but the actual method of reading `todos` may change in the future.

With this in mind, we are going to update our root reducer with a named selector export. It will also be called `getVisibleTodos`, and like before it also accepts `state` and `filter`. However, in this case `state` corresponds to the state of the combined reducer.

Now we want to be able to call the `getVisibleTodos` function defined in the `todos` file alongside the reducer, but we can't use a named import because there is a function with exactly the same name in the scope.

To work around this, we will use the name space import syntax that puts all the exports on an object (called `fromTodos` in this case).

Now we can use `fromTodos.getVisibleTodos()` to call the function we defined in the other file, and pass the slice of the `state` corresponding to the `todos`.

#### Root Reducer Update (`src/reducers/index.js`)
```javascript
import { combineReducers } from 'redux';
import todos, * as fromTodos from './todos';

const todoApp = combineReducers({
  todos,
});

export default todoApp;

export const getVisibleTodos = (state, filter) =>
  fromTodos.getVisibleTodos(state.todos, filter);
```

### Updating `VisibleTodoList`
Now we can go back to our `VisibleTodoList` component and import `getVisibleTodos` from the root reducer file.

`import { getVisibleTodos } from '../reducers'`

`getVisibleTodos` encapsulates all the knowledge about the application state shape, so we can just pass it the whole state of our application and it will figure out how to select the visible todos according to the logic described in our selector.

Finally, we can refer back to our `mapStateToProps` mapping function and call `getVisibleTodos()` with the entire state, rather than with `state.todos` like we used to (this logic has been moved to the reducer):

```
const mapStateToProps = (state, { params }) => ({
      todos: getVisibleTodos(state, params.filter || 'all' )
  });
```

_Note: the explanations in the video require a version of `react-router` previous to the 4.0.0. Starting in that version some changes have been included which require this slightly different syntax:_

### react-router v4.0.0 or superior
```
const mapStateToProps = (state, { match }) => ({
      todos: getVisibleTodos(state, match.params.filter || 'all' )
  });
```

[Recap at 2:51 in video](https://egghead.io/lessons/javascript-redux-colocating-selectors-with-reducers?series=building-react-applications-with-idiomatic-redux)


<p align="center">
<a href="./09-Using_mapDispatchToProps_Shorthand_Notation.md"><- Prev</a>
<a href="./11-Normalizing_the_State_Shape.md">Next -></a>
</p>