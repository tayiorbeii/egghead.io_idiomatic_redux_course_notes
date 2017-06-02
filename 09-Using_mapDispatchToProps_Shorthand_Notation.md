# 09. Using `mapDispatchToProps()` Shorthand Notation
[Video Link](https://egghead.io/lessons/javascript-redux-using-mapdispatchtoprops-shorthand-notation)

The `mapDispatchToProps` function lets us inject certain props into the React component that can dispatch actions. For example, the `TodoList` component calls its `onTodoClick` callback prop with the `id` of the `todo`.

#### Inside `TodoList`
```javascript
      <Todo
        key={todo.id}
        {...todo}
        onClick={() => onTodoClick(todo.id)}
      />
```

Inside `mapDispatchToProps` in our `VisibleTodoList` component we specify that when `onTodoClick()` is called with an `id`, we want to dispatch the `toggleTodo` action with this `id`. The `toggleTodo` action creator uses this `id` to generate an action object that will be dispatched.

#### `VisibleTodoList` `mapDispatchToProps`
```javascript
const mapDispatchToProps = (dispatch) => ({
  onTodoClick(id) {
    dispatch(toggleTodo(id));
  },
});

const VisibleTodoList = withRouter(connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList));
```

When the arguments for the callback prop match the arguments to the action creator exactly, there is a shorter way to specify `mapDispatchToProps`.

Rather than pass a function, we can pass an object mapping of the names of the callback props that we want to inject and the action creator functions that create the corresponding actions.

This is a rather common case, so often you don't need to write `mapDispatchToProps`, and you can pass this map in object instead.

#### `VisibleTodoList` After:
```javascript
const VisibleTodoList = withRouter(connect(
  mapStateToProps,
  { onTodoClick: toggleTodo }
)(TodoList));
```

[Recap at 1:13 in video](https://egghead.io/lessons/javascript-redux-using-mapdispatchtoprops-shorthand-notation)

<p align="center">
<a href="./08-Using_withRouter_to_Inject_the_Params_into_Connected_Components.md"><- Prev</a>
<a href="./10-Colocating_Selectors_with_Reducers.md">Next -></a>
</p>