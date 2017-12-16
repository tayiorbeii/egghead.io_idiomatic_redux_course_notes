# 07. Filtering Redux State with React Router Params
[Video Link](https://egghead.io/lessons/javascript-redux-filtering-redux-state-with-react-router-params)

Now we're using the `Link`s provided by React Router, so when we click a link the URL gets updated. However, the content doesn't get updated because the visible `TodoList` component in its `mapStateToProps` function still depends on the `visibilityFilter` in the Redux store instead of reading it from the URL.

#### `mapStateToProps` Before:
```javascript
const mapStateToProps = (state) => ({
  todos: getVisibleTodos(
    state.todos,
    state.visibilityFilter
  )
})
```

In order to fix this, we'll add an argument `ownProps`, and we'll read the `visibilityFilter` from `ownProps`.

#### `mapStateToProps` After:
```javascript
const mapStateToProps = (state, ownProps) => ({
  todos: getVisibleTodos(
    state.todos,
    ownProps.filter
  )
})
```

We will also update our `getVisibleTodos` function to use our current convention for filter props `'all'`, `'completed'`, and `'after'`.

#### `getVisibleTodos` Before:
```javascript
const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case 'SHOW_ALL':
      return todos;
    case 'SHOW_COMPLETED':
      return todos.filter(t => t.completed);
    case 'SHOW_ACTIVE':
      return todos.filter(t => !t.completed);
    default:
      throw new Error(`Unknown filter: ${filter}.`);
  }
};
```

#### `getVisibleTodos` After:
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
```

## Updating `App.js`

Since the `VisibleTodoList` component gets rendered from the app, we need to add the `filter` prop to make it available in the `mapStateToProps` function of the `VisibleTodoList`.

We want the filter prop to correspond to the current `filter` parameter in our route configuration (the `path='/(:filter)'` we set in a previous lesson). React Router makes these parameters available to the route handler component in a special prop called `params`, so we will add `params` as a prop to `App`. Now we will be able to read the filter from `params.filter`.

Since the `filter` param is empty on the root path, we'll pass `'all'` to `VisibleTodoList` as a fallback.

```javascript
const App = ({ params }) => (
  <div>
    <AddTodo />
    <VisibleTodoList
      filter={params.filter || 'all'}
    />
    <Footer />
  </div>
);
```

## `App.js` (react-router v4.0.0 or superior)
```javascript
const App = ({ match }) => (
  <div>
    <AddTodo />
    <VisibleTodoList
      filter={match.params.filter || 'all'}
    />
    <Footer />
  </div>
);
```


Now that our visibility filters are managed by React Router, we no longer need the `visibilityFilter` reducer. We can delete it, and also remove it from the `combineReducers()` declaration in `index.js`.


[Recap at 2:20 in video](https://egghead.io/lessons/javascript-redux-filtering-redux-state-with-react-router-params)


<p align="center">
<a href="./06-Navigating_with_React_Router_Link.md"><- Prev</a>
<a href="./08-Using_withRouter_to_Inject_the_Params_into_Connected_Components.md">Next -></a>
</p>
