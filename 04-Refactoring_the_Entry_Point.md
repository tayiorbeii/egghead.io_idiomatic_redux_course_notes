# 04. Refactoring the Entry Point
[Video Link](https://egghead.io/lessons/javascript-redux-refactoring-the-entry-point?series=building-react-applications-with-idiomatic-redux)

In this lesson we will extract the logic necessary for creating & subscribing to the store into a separate file.

#### `index.js` Before
```javascript
import 'babel-polyfill'
import React from 'react'
import { render } from 'react-dom'
import { Provider } from 'react-redux'
import { createStore } from 'redux'
import throttle from 'lodash/throttle'
import todoApp from './reducers'
import App from './components/App'
import { loadState, saveState } from './localStorage'

const persistedState = loadState()
const store = createStore(
  todoApp,
  persistedState
);

store.subscribe(throttle(() => {
  saveState({
    todos: store.getState().todos,
  })
}, 1000))

render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```

We will call our new file `configureStore.js`, and start by creating a funcion `configureStore` that will contain the store creation & persistence logic.

We do this because this way our app doesn't have to know exactly how the store is created and whether or not we have subscribe handlers. It can just use the returned store in the `index.js` file.

#### `configureStore.js`
```javascript
import { createStore } from 'redux'
import throttle from 'lodash/throttle'
import todoApp from './reducers'
import { loadState, saveState } from './localStorage'

const configureStore = () => {
  const persistedState = loadState()
  const store = createStore(todoApp, persistedState)

  store.subscribe(throttle(() => {
    saveState({
      todos: store.getState().todos
    })
  }, 1000))

  return store
}

export default configureStore
```

By exporting `configureStore` instead of just `store`, we will be able to create as many store instances as we want for testing.

#### `index.js` After
```javascript
import 'babel-polyfill'
import React from 'react'
import { render } from 'react-dom'
import Root from './components/Root'
import configureStore from './configureStore'

const store = configureStore()
render(
  <Root store={store} />,
  document.getElementById('root')
);
```

Note that we have also extracted the root rendered element into a separate component called `Root`. It accepts `store` as a prop, and will be defined in a separate file in our `src/components` folder.

---

#### `Root` Component
```javascript
import React, { PropTypes } from 'react';
import { Provider } from 'react-redux';
import App from './App';

const Root = ({ store }) => (
  <Provider store={store}>
    <App />
  </Provider>
);

Root.propTypes = {
  store: PropTypes.object.isRequired,
};

export default Root;
```

We've defined a stateless functional component that just takes the `store` as a prop and returns `<App />` inside of `react-redux`'s `Provider`.

Now inside of `index.js`, we can remove our `Provider` import as well as replacing the `App` component import with our `Root` component import.

[Recap at 1:45 in video](https://egghead.io/lessons/javascript-redux-refactoring-the-entry-point?series=building-react-applications-with-idiomatic-redux#/tab-transcript)


<p align="center">
<a href="./03-Persisting_the_State_to_the_Local_Storage.md"><- Prev</a>
<a href="./05-Adding_React_Router_to_the_Project.md">Next -></a>
</p>
