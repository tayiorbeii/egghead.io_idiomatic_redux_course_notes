# 13. Adding a Fake Backend to the project
[Video Link](https://egghead.io/lessons/javascript-redux-adding-a-fake-backend-to-the-project)

In the next lessons, we won't be dealing with persistence anymore. Instead, we're going to add some asynchronous fetching to the app.

We are now removing all the code related to the `localStorage` persistence, and deleting the `localStorage.js` file as well, because a new module has been added that implements a fake remote API.

All of the todos are kept in memory, and an artificial delay has been added. We also have methods that return Promises just like a real API implementation.

#### `src/api/index.js`
```javascript
import { v4 } from 'node-uuid';

// This is a fake in-memory implementation of something
// that would be implemented by calling a REST server.

const fakeDatabase = {
  todos: [{
    id: v4(),
    text: 'hey',
    completed: true,
  }, {
    id: v4(),
    text: 'ho',
    completed: true,
  }, {
    id: v4(),
    text: 'let’s go',
    completed: false,
  }],
};

const delay = (ms) =>
  new Promise(resolve => setTimeout(resolve, ms));

export const fetchTodos = (filter) =>
  delay(500).then(() => {
    .
    .
    .
```

This approach lets us explore how Redux works with asynchronous data fetching without writing a real backend for the app.

Now we will import `fetchTodos` into the different modules of our app.

`import { fetchTodos } from './api'`

We will learn how to put these todos into the Redux store later, but for now, we can call `fetchTodos` with a filter argument that will return a Promise that resolves through an array of todos, just like a REST backend would return an array.

#### `index.js`
```javascript
fetchTodos('all').then(todos =>
  console.log(todos)
);
```

The fake API waits for half a second to simulate the network connection, and then resolves the promise to an array of todos that we will treat as if they were retrieved from a remote server.


<p align="center">
<a href="./12-Wrapping_dispatch_to_Log_Actions.md"><- Prev</a>
<a href="./14-Fetching_Data_on_Route_Change.md">Next -></a>
</p>