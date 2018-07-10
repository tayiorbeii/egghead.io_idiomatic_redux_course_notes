# 21. Displaying Loading Indicators
[Video Link](https://egghead.io/lessons/javascript-redux-displaying-loading-indicators)

When the data is fetched asynchronously, we want to display some kind of visual cue to the user. We'll add a condition to the `render` function that says that if we're fetching data, and we have no to-dos to show, we'll return a loading indicator from the `render` function of `VisibleTodoList`.

We will grab `todos` and `isFetching` from `props`. Since `todos` is the only extra prop we need to pass to the list, instead of using the spread operator, we will just pass the `todos` directly.

Our `mapStateToProps` function already calculates `visibleTodos` and includes the `todos` in the props. We need to do something similar for `isFetching`. `getIsFetching` accepts the current `state` of the app, and the `filter` for the `todos` being fetched. It is declared alongside other top level selectors in the top level reducer file.

#### Inside `VisibleTodoList.js`
```javascript
render() {
    const { isFetching, toggleTodo, todos } = this.props;
    if (isFetching && !todos.length) {
      return <p>Loading...</p>;
    }

    return (
      <TodoList
        todos={todos}
        onTodoClick={toggleTodo}
      />
    );
  }

  /// PropType Declarations...

  const mapStateToProps = (state, { params }) => {
    const filter = params.filter || 'all';
    return {
      isFetching: getIsFetching(state, filter),
      todos: getVisibleTodos(state, filter),
      filter,
    };
  };
```

### Updating the Root Reducer

Switching to our root reducer file (`src/reducers/index.js`) we will add another exported named selector function for `getIsFetching`. It accepts the `state` and the `filter` as arguments, and it delegates to another selector to find if the list is currently being fetched.

We will pass in the state of this list from `state.listByFilter`, but we haven't written `getIsFetching` yet.

```javascript
export const getIsFetching = (state, filter) =>
  fromList.getIsFetching(state.listByFilter[filter]);
```

### Updating `createList.js`

Before creating our new `getIsFetching` selector, we need to modify the state shape of the list. Rather than assume `state` is an array of `id`s, we will assume it is an object that contains this array as a property.

Now we can add another selector called `getIsFetching` that reads `state.isFetching`.

```javascript
export const getIds = (state) => state.ids;
export const getIsFetching = state => state.isFetching;
```

We want our reducer to keep track of both of these fields, so rather than complicate the existing `createList` reducer, we will rename it to `ids`, because it manages just the `id`s.

### Creating `isFetching`

First, at the top of our file we need to add an import for the `combineReducers` utility from Redux:
```javascript
import { combineReducers } from 'redux';
```

Now we can create our `isFetching` reducer that will manage just the `state`'s `isFetching` flag.

Its initial state is `false`, and it looks just like any other reducer. We switch on the `action` `type`, and if it is `'REQUEST_TODOS'`, we'll return `true` because we started fetching.

If it's `'RECEIVE_TODOS'`, we'll return `false` because the operation has finished. For any unknown action, we'll return the current `state`.

We'll include the same condition as our `ids` reducer that ignores any actions with a filter that does not match the `filter` argument to `createList`.

#### Inside `createList.js`
```javascript
const isFetching = (state = false, action) => {
    if (filter !== action.filter) {
      return state;
    }
    switch (action.type) {
      case 'REQUEST_TODOS':
        return true;
      case 'RECEIVE_TODOS':
        return false;
      default:
        return state;
    }
  };
```

Notice that we handle the `REQUEST_TODOS` action, but we are not dispatching it anywhere.

### Adding the 'REQUEST_TODOS' Action Creator

In our action creators file (`src/actions/index.js`), we will add a new exported function called `requestTodos` that takes the `filter` and returns an action object describing the `'REQUEST_TODOS'` action with the corresponding filter.

```javascript
export const requestTodos = (filter) => ({
  type: 'REQUEST_TODOS',
  filter,
});
```

Every exported action creator will be available on the `props` of the `VisibleToDoList` component.

### Updating `fetchData` inside `VisibleToDoList`
We can destructure `requestTodos` from the `props`, and call it right before starting the asynchronous `fetchToDos` operation.

```javascript
fetchData() {
  const { filter, fetchTodos, requestTodos } = this.props;
  requestTodos(filter);
  fetchTodos(filter);
}
```

[Recap at 3:51 in video](https://egghead.io/lessons/javascript-redux-displaying-loading-indicators)


<p align="center">
<a href="./20-Refactoring_the_Reducers.md"><- Prev</a>
<a href="./22-Dispatching_Actions_Asynchronously_with_Thunks.md">Next -></a>
</p>
