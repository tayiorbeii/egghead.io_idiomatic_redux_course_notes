# 08. Using `withRouter()` to Inject the Params into Connected Components
[Video Link](https://egghead.io/lessons/javascript-redux-using-withrouter-to-inject-the-params-into-connected-components)

Currently we are reading `params.filter` passed by the Router inside of the `App` component. We can access the params from there because the router injects the `params` prop into any Route handler component specified in the route configuration. In this case, the `filter` is passed inside `params`.

However, the `App` component itself doesn't use `filter`, it just passes it to `VisibleTodoList`, which uses it to calculate the currently visible Todos.

Passing the `params` from the top level of Route Handlers gets tedious, so we'll remove the `filter` prop. Instead, we'll find a way to read the current Router `params` in the `VisibleTodoList` itself.

#### `App.js` Before:
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

#### `App.js` After:
```javascript
const App = () => (
  <div>
    <AddTodo />
    <VisibleTodoList />
    <Footer />
  </div>
);
```

## Updating `VisibleTodoList`
We will start by importing `withRouter` from React Router (version 3+):

`import { withRouter } from 'react-router'`

`withRouter` takes a React component and returns a different React component that injects the router-related props such as `params` into your component.

We want `params` to be available inside of `mapStateToProps`, so we need to wrap the `connect()` result so that the connected component gets the `params` as a prop.

We will also need to change `mapStateToProps` to read the `filter` from `ownProps.params` instead of `ownProps` directly.

Like before, we'll specify the fallback value to `'all'`.

#### `VisibleTodoList` Before:
```javascript
const mapStateToProps = (state, ownProps) => ({
  todos: getVisibleTodos(
    state.todos,
    ownProps.filter
  ),
});

// mapDispatchToProps ...

const VisibleTodoList = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList);
```

#### `VisibleTodoList` After:
```javascript
const mapStateToProps = (state, ownProps) => ({
  todos: getVisibleTodos(
    state.todos,
    ownProps.params.filter || 'all'),
});

// mapDispatchToProps ...

const VisibleTodoList = withRouter(connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList));
```

We can make `mapStateToProps` even more compact by reading `params` directly from the argument definition thanks to the ES6 destructuring syntax:

```javascript
const mapStateToProps = (state, { params }) => ({
  todos: getVisibleTodos(state.todos, params.filter || 'all'),
});
```

_Note: the explanations in the video require a version of `react-router` previous to the 4.0.0. Starting in that version some changes have been included which require this slightly different syntax:_

#### `VisibleTodoList` After (react-router v4.0.0 or superior):
```javascript
const mapStateToProps = (state, ownProps) => ({
  todos: getVisibleTodos(
    state.todos,
    ownProps.match.params.filter || 'all'),
});

// mapDispatchToProps ...

const VisibleTodoList = withRouter(connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList));
```

We can make `mapStateToProps` even more compact by reading `params` directly from the argument definition thanks to the ES6 destructuring syntax:

```javascript
const mapStateToProps = (state, { match }) => ({
  todos: getVisibleTodos(state.todos, match.params.filter || 'all'),
});
```

[Recap at 1:51 in video](https://egghead.io/lessons/javascript-redux-using-withrouter-to-inject-the-params-into-connected-components#/tab-transcript)


<p align="center">
<a href="./07-Filtering_Redux_State_with_React_Router_Params.md"><- Prev</a>
<a href="./09-Using_mapDispatchToProps_Shorthand_Notation.md">Next -></a>
</p>