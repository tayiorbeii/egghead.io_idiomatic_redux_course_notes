# 05. Adding React Router to the Project
[Video Link](https://egghead.io/lessons/javascript-redux-adding-react-router-to-the-project?series=building-react-applications-with-idiomatic-redux)

In this lesson, we will add React Router.

#### `Root.js` Before
```javascript
import React, { PropTypes } from 'react';
import { Provider } from 'react-redux';
import App from './App';

const Root = ({ store }) => (
  <Provider store={store}>
    <App />
  </Provider>
);
.
.
.
```

To add React Router to the project, run:
`$ npm install --save react-router`

Inside of `Root.js` we will import the `Router` and `Route` components.

We also replace our `<App />` with `<Router />`. It's important that it is still inside of `<Provider />` so that any components rendered by the router still have access to the store.

Inside of `<Router />` we will put a single `<Route />` element that tells React Router that we want to render our `<App />` component at the root path (`'/'`) in the browser's address bar.

#### `Root.js` After
```javascript
import React, { PropTypes } from 'react';
import { Provider } from 'react-redux';
import { Router, Route, browserHistory } from 'react-router';
import App from './App';

const Root = ({ store }) => (
  <Provider store={store}>
    <Router history={browserHistory}>
      <Route path="/" component={App} />
    </Router>
  </Provider>
);
.
.
.
```

_Note: the video contains a fix for weird address bar symbols stemming from an old release of `react-router`_

_Note: the explanations in the video require a version of `react-router` previous to the 4.0.0. Starting in that version some changes have been included which require this slightly different syntax:_

#### `Root.js` After (react-router v4.0.0 or superior)
```javascript
import React, { PropTypes } from 'react';
import { Provider } from 'react-redux';
import App from './App';
import { BrowserRouter, Route } from 'react-router-dom'

const Root = ({ store }) => (
    <Provider store={store}>
        <BrowserRouter>
            <Route path="/" component={App} />
        </BrowserRouter>
    </Provider>
);
.
.
.
```

[Recap at 1:09 in video](https://egghead.io/lessons/javascript-redux-adding-react-router-to-the-project?series=building-react-applications-with-idiomatic-redux#/tab-transcript)


<p align="center">
<a href="./04-Refactoring_the_Entry_Point.md"><- Prev</a>
<a href="./06-Navigating_with_React_Router_Link.md">Next -></a>
</p>