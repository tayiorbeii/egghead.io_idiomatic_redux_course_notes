# 01. Simplifying the Arrow Functions
[Video Link](https://egghead.io/lessons/javascript-redux-simplifying-the-arrow-functions?course=building-react-applications-with-idiomatic-redux)

Since action creators are just regular JavaScript functions, you can define them any way you like.

For example, if you don't like arrow notation, you can replace it with the traditional function declaration syntax:

##### Arrow Function Syntax
``` javascript
export const addTodo = (text) => {
  return {
    type: 'ADD_TODO',
    id: (nextTodoId++).toString(),
    text,
  };
};
```
##### Traditional Function Syntax
``` javascript
export function addTodo(text) {
  return {
    type: 'ADD_TODO',
    id: (nextTodoId++).toString(),
    text,
  };
}
```

However, if you like using arrow functions, they can be made even more concise.


Looking at our arrow function above, we can see that our function starts and ends with a curly brace, and contains a `return` statement inside. Since the return statement is all that is inside our function, we can use it as the body of the arrow function.

We can remove the block in favor of the object expression:
```javascript
export const addTodo = (text) => ({
  type:'ADD_TODO',
  id: (nextTodoId++).toString(),
  text,
})
```

*Note:* It is important to wrap the expression in parents so that the parser understands this as an expression instead of a block.

These steps can be repeated for any function that just returns an object; just remove the `return` statement, and change the body into an expression.

This process can be used outside of Action Creators as well. For example, it is common for `mapStateToProps` and `mapDispatchToProps` to just return objects:
##### Before:
```javascript
const mapStateToProps = (state, ownProps) => {
  return {
    active: ownProps.filter === state.visibilityFilter
  }
}

const mapDispatchToProps = (dispatch, ownProps) => {
  return {
    onClick: () => {
      dispatch(setVisibilityFilter(ownProps.filter))
    }
  }
}
```

##### After:
```javascript
const mapStateToProps = (state, ownProps) => ({
    active: ownProps.filter === state.visibilityFilter
})

const mapDispatchToProps = (dispatch, ownProps) => ({
    onClick: () => {
      dispatch(setVisibilityFilter(ownProps.filter))
    }
})
```

We can make `mapDispatchToProps` even more compact by replacing the arrow function with a concise method notation that is part of ES6 and is available when a function is defined inside an object.

```javascript
const mapDispatchToProps = (dispatch, ownProps) => ({
    onClick() {
      dispatch(setVisibilityFilter(ownProps.filter))
    }
})
```

#### [Recap at 2:05 in video](https://egghead.io/lessons/javascript-redux-simplifying-the-arrow-functions?course=building-react-applications-with-idiomatic-redux)

<p align="center">
<a href="./02-Supplying_the_Initial_State.md">Next -></a>
</p>
